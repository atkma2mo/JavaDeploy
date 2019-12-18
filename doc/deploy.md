# JavaDeploy  


Jenkins CircleCIなどは使いません。学内Webサーバへのデプロイ手順を説明します。古き良きサーバの構築をします。  

# ネットワーク図  


ローカルの開発環境は  
・Openjdk11  
・Spring boot(Tomcat) 
・Wicket  
・Maven  
・h2DB  
・InteliJ  
を使用する。  

デプロイ先のサーバには  
・Openjdk11  
・Maven  
・h2DB  
・Nginx  
・Debian
をリモートサーバにインストールする。  


![無題のプレゼンテーション (3)](https://user-images.githubusercontent.com/44164993/71048825-e97df580-2183-11ea-8103-3941949fd087.png)  


  

http://www.nsrg.fml.org/notes/proj-2018/


## Debianインストール  

http://www.fml.org/home/fukachan/ja/debian.install.html  


## warファイルを作る  

※Windowsでは文字コードの問題で不具合が発生する場合があるため、注意する。  

------------------------------------------------  

mvn clean packeageでコマンドで作るのを推奨  

------------------------------------------------  

まずはIntelliJでwarファイルを生成する。  

IntelliJで実行可能なJavaEEアプリケーションのプロジェクトを開き、File -> Project Structureを開く。  

左メニューのArtifactsを選択する。  

画面上部にある「＋」ボタンを選択して、Web Application: Archive -> For ‘web:war exploded’を選択する。  

すると、web:warというArtifactsが追加さるので、ＯＫボタンを押して閉じる。  

今度はBuild -> Build Artifactsを選択する。  

小さなウィンドウが表示されるので、web:war -> Buildを選択します。するとwarファイルの生成が始まりまる。  

warファイルの生成に成功すると、画面左のプロジェクトメニューの、{プロジェクト名} -> out -> artifacts -> web_warの下にweb_war.warというwarファイルが表示されるので、生成できた事がわかる。



https://kokichiblog.com/it/800/  



## サーバセキュリティの設定(最低限)  


セキュリティについては必要なものを吟味してください。  

  
Debian環境で進めます。

Debian 

https://www.rem-system.com/debian10-first-settings/  


https://www.server-world.info/query?os=Debian_10  


http://archiva.jp/web/server-side/debian-setup.html    


CentOS前半部分  

https://qiita.com/tadatti/items/9f4911c624aafba63258  


http/https Let,s Encryptとかもいる。  


## 初期インストールされたパッケージをアップデート  

※最新版になるため注意  

```$ apt update``` 　　


```$ apt upgrade```  
   
rootログイン。パスワードが求められるのでログインする。  

```$ su```  
  
rootのパスワードを入力する。  
 

```Password:```  


rootユーザから一般ユーザへの切り替え  
```#exit```  
実行するとプロンプトが$に変わるのを確認する。  

## vim backspaceを使えるようにする。  


「.vimrc」ファイルに  

```set backspace=indent,eol,start```  


## nviの使いかた  

Escを押したあと、  
a、iで挿入モード  
oで改行  
x     1文字削除  
dd    1行削除  
yy    1行コピー  
nyy   n行コピー  
p     カーソルの文字の次または次の行にペースト  
P     カーソルの文字の前または前の行にペースト  
u     カット、ペーストを1回取消し  

カーソル移動  
    上 k  
    下 j  
    右 l  
    左 h  

カレント行の行頭へ移動  
    0  
    
:[数字]で数字の行へ移動する  

gg  で1行目へ移動できる  
G   で最終行へ移動できる  

# Openjdk11 maven h2DB nginxをインストール  

今回はSpring bootに組み込まれているTomcatを使うため、リモートサーバにはTomcatをインストールしない。  

今回とは関係ないが参考として、Apache、Tomcat、jdkのインストール設定  
https://qiita.com/Uejun/items/adb428cccf96f8509b8f  

https://centossrv.com/tomcat9.shtml  

https://yulii.github.io/nginx-tomcat-proxy-20110523.html  


↑Apache or NginxとTomcatの連携を行う必要がある。TomcatはAPサーバだが、Apache or Nginxを除いたTomcat単体でもWebサーバとしての機能を確保している。Tomcatだけでサーバ処理を行えばシンプルだが、Apache or Nginxと比較すれば性能は劣るため、2つを連携させることでより優位なWebサーバ構築が可能となる。(Apache Tomcat連携で調べる。または、AJP Nginx Tomcat 連携。Nginx Tomcat リバースプロキシ。)  


ApacheとNginxのメリット、デメリットは↓  
https://qiita.com/kamihork/items/49e2a363da7d840a4149  



前提条件  
Debianシステムにパッケージをインストールできるようにするには、sudo特権を持つユーザーとしてログインする必要がある。  

```$ su```    


パッケージの更新  

```$ apt install ```  


インストール時にはtmpファイルで行ってあとで消せるようにする。  

```cd /tmp```  



## Openjdk11のインストール  

Openjdk11のインストールをする。  
```# apt install openjdk-11-jdk```    

システムに複数のJavaバージョンがインストールされている場合は、次のコマンドを使用してバージョンを切り替える。  

```# update-alternatives --config java```  

システムが正しいJDKを使用していることを確認する。  

```# java -version```  
 

zuluとか使わないほうがいい。    
default-jdkも誤作動あるかもなのでフルパスで指定。  


Openjdkサイト  
https://openjdk.java.net/  

## Apache mavenのインストール 

今なら3.6.3(2019/12/06現在)  

Mavenをインストール  

```# apt install maven ```  

[Y/n]でYを入力し、Enter  

  

環境変数に設定されているパスを表示する。  



メモする。

```# echo JAVA_HOME```  

```# echo M2_HOME```   

```# echo MAVEN_HOME```  
  

環境変数のセットアップ    

```# vi /etc/profile.d/maven.sh```  

  
```
export JAVA_HOME=[echoしたJAVA_HOME] 
export M2_HOME=[echoしたM2_HOME]  
export MAVEN_HOME=[echoしたMAVEN_HOME]  
export PATH=${M2_HOME}/bin:${PATH}   
```  

ファイルを保存して閉じる。  
```:wq!```  

chmodコマンドを入力して、スクリプトを実行可能にする。  

```# chmod +x /etc/profile.d/maven.sh```  

sourceコマンドを使用して環境変数をロードする。

```# source /etc/profile.d/maven.sh```  


mavenインストールの確認  

```# mvn -version```  



mavenダウンロードページ  
https://maven.apache.org/download.cgi   




## h2DBのインストール  

h2DBはPureJavaのため、事前にjdkのインストールが必要です。先ほどインストールしたので大丈夫。    

```# wget http://www.h2database.com/h2-2017-03-10.zip```  

2019/12/06現在  

```# unzip h2-2019-10-14.zip```  

```# cd h2```  

webAllowOthers=trueはremote connectionを有効にするための設定  

```# echo webAllowOthers=true > ~/.h2.server.properties```  

```# java -jar bin/h2-1.4.194.jar```  

1.4.200  

アクセス  

http://localhost:8282  
にアクセスするとログイン画面がでる。  

JDBC:URLに  

サーバモードは  

```jdbc:h2:tcp://localhost/~/データベース名```  

組み込みモード(今回)  

```jdbc:h2:~/データベース名```  

と入力して接続テストを行い、successとなれば接続する。  

 
h2DBのダンプファイルをつくり、リストアする  


inteliJのデータベース->右クリック->ファイルにデータをダンプ
->  

リモートサーバ内にファイル送ってから、結局SQLをコピーしてはった。  


h2DBの設定方法  
http://yoks.blue.coocan.jp/TechNote/H2/H2_ope2.htm  


h2DB公式  
http://www.h2database.com/html/tutorial.html#console_settings 
 　　

## nginxのインストール  

キーを作る。  

```# wget http://nginx.org/keys/nginx_signing.key```   

```# apt-key add nginx_signing.key```  

viでファイルを開く。  

```#vi /etc/apt/sources.list```  

ファイルの最終行に以下2行を書き込む。(安定板)  


deb http://nginx.org/packages/debian/ stretch nginx  
deb-src http://nginx.org/packages/debian/ stretch nginx  

nginxのインストール  


```# apt -y install nginx```  

nginxのパッケージが足りないと言われることがあるので、

```apt update```  
して、依存関係がないようにapt install nginx をもう一度する。  


インストールの確認 -Vで詳細情報の表示  

```# nginx -V```  


nginx -v として実行されるものはプログラム本体であり、それはパスが通っているディレクトリから探される。つまり、echo $PATH で表示されるディレクトリのいずれかである。  

which nginx とすることで、ディレクトリを指定しないときに実行される nginx コマンドの位置が表示される。これは上記の PATH に含まれているいずれかのディレクトリになる。

/etc/nginx 以下のファイルは設定ファイルです。
動作がおかしい場合はこのディレクトリ内の設定ファイルが誤っていることが多い。

自動起動の設定(インストール直後は自動起動になっているので無視してよい)。  

「/etc/init.d」は、デーモンなどの起動スクリプトが設置されているディレクトリです。  
Red Hat系のLinuxディストリビューションでは「serviceコマンド」を利用してデーモンの起動・停止・再起動を行うことが多いので、実行した覚えがないという方もおられるかもしれませんね。  


```# /etc/init.d/ enable nginx```  

```# /etc/init.d/ is-enabled nginx```  


自動起動をやめる  

```# /etc/init.d disable nginx ```  

```# /etc/init.d is-enabled nginx```  

nginxにリバースプロキシの設定を追加  


nginx 80/tcp で受けて 127.0.0.1:8080/tcp へ転送
/etc/nginx/sites-enabled/reverse-proxy.conf を作成(注意: root権限が必要。su ...)
/etc/nginx/sites-enabled/reverse-proxy.conf の中身は、こんなかんじ↓

```
server {  
	location / {
            proxy_pass http://localhost:8080;
        }
}
```

proxy_pass実行するURL  
  
8080はTomcat  
  
  
設定ファイルを変更したら、リロードを忘れないようにね。
```
#/etc/init.d/nginx reload
```

https://www.mk-mode.com/blog/2017/09/16/debian-9-nginx-installation-by-official-apt/  

nginx公式  
https://nginx.org/en/


# scpコマンドでデプロイ  

ssh入ってなかったのでいれる。  

```# apt install ssh```  

https://www.karelie.net/install-ssh-ubuntu-1804/

vncを同一パソコンでひらく

```scp warファイル username@172.xx.○○○.□□□:△```  
△より後ろにパス指定しないとデフォでリモートのhome下に設定される。  

 □□□:△をip addrで探す。
 enp0s3?(VMにおけるインターフェース名)のところを置き換える。100だったら。  
 
 ```scp warファイル username@172.xx.○○○.100```


# warの実行  

warファイルはローカルのPCからGoogleDriveなどでリモートに共有して渡す。


```java -war /home/proj/project-de-build-sitamono.war --server.port=8080 &```  

&は裏でずっと実行する。  

javaClassNotFoundExeption、Java環境で開発したグループが全部なっている。

pomの設定で解決できる。timestampの影響らしい。wicket spring bootのどこでやるからしい。  
自分はかかわっていないので詳しくはわかりません。

h2DBのリモートの接続でつまづいた。  

jdbc:h2./dataとかwebconsoleでパスの設定をapplicationpropertiesと同じにするうんぬんが困惑した。  
一回DBのtable消えたりしたから意味不明。  
2回目はできて成功した。pathをどうにかって感じ。  


再起動は
```/sbin/shutdown -n```

sbinはシステム  


理想はshellscript書いて自動化ができれば…  




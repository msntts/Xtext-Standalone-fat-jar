# Xtext-Standalone-fat-jar
Xtextプロジェクトから、コード生成をするRunnableなfat jarを生成するための設定方法について記述しています。

## 動作確認環境
この環境ではないと出来ないというわけではないと思いますが、特にeclipseは見た目が変わっていたりするかもしれないので一応書いておきます。  

| 名前         | バージョン       |
| ---          | ---              |
| OS           | Windows 11       |
| eclipse      | 2021-09 (4.21.0) |
| Xtext Plugin | 2.254.0          |

以降の説明では、eclipseからXtextプロジェクトを生成した後の操作を記述しています。プロジェクト名はデフォルトの```org.xtext.example.mydsl```を使っています。  

## Runnable Jarにするための準備
[Git Diff](https://github.com/msntts/Xtext-Standalone-fat-jar/commit/17235435ab4774dab62727cee35e2464e9bb6197)  
```org.xtext.example.mydsl```プロジェクトの```src/org/xtext/example/mydsl/GenerateMyDsl.mwe2```に、JavaのMain関数を生成するオプションを追加します。

```
// 省略
generator = {
	generateXtendStub = true
	generateJavaMain = true // add
}
```

その後、MWE2ワークフローを実行すると、```src/org/xtext/example/mydsl/generator/Main.java```が生成されます。

## eclipseにJar Export用の設定を追加
[Git Diff](https://github.com/msntts/Xtext-Standalone-fat-jar/commit/c0bd15bfa5ba313f6113613cafa745ac62d18c51)  
JarのExportのため、MainクラスやExportするプロジェクトの設定を行います。  

- eclipseのtoolバー > Run > Run Configuration を選択
- Javaアプリケーションを選択し、New launch Configurationをクリック
- Nameに適当な名前を入力
  - 本リポジトリでは、```run generator```
- MainタブのProjectに先ほど生成した```Main.java```のあるプロジェクトを選択
  - 本リポジトリでは、```org.xtext.example.mydsl```
- MainタブのMain classにMain.javaのフル修飾名を入力
  - 本リポジトリでは、```org.xtext.example.mydsl.generator.Main```
- Applyボタンを押し、画面を閉じる

## Fat Jarのエクスポート
- eclipseのtoolバー > File > Exportを選択
- Java > Runnable JAR Fileを選択し、Nextをクリック
- Launch configurationsに先ほど作成したJavaアプリケーションのlanunch configurationを選択
  - 本リポジトリでは、```run generator```
- Export destinationに任意のフォルダ&Jarファイル名を入力
  - 本リポジトリでは```<project-root>/mydslGenerator.jar```
- Library handringは```Package required libraries into generated JAR```を選択
- (オプション)Save as ANT Scriptにチェックを入れ、ANTスクリプトの生成先と名前を入力
  - この時ANTスクリプトは、生成したフォルダからの相対パスで処理が生成されるので、ANTスクリプトを移動した場合はパスの書き換えが必要
- Finishボタンをクリック

 ## Fat Jarの実行
 デフォルトのMain.javaでは、第一引数で受けたmydslファイルに対してコード生成を実行します。  
 実行は、Javaのパスが通っているコマンドプロンプトにて
 ```sh
java -jar mydslGenerator.jar <path>/<to>/mydsl.mydsl
 ```
 を実行すると、src-genフォルダ以下に、コード生成結果が出力されます。

 ## コマンドライン実行
 Fat Jarのエクスポート時にANTスクリプトの生成をしている場合、コマンドラインから
 ```sh 
 ant <ANTスクリプト名>
 ```
 でFat Jarを生成できるようになります。  
 もし、参照するプラグインやライブラリを追加したときは、サイドEclipseからFat Jarの生成を行ってANTスクリプトを更新してください。  
 また、eclipseのプロジェクトにANTスクリプトを追加した場合は
 
 -  eclipse上からANTスクリプトを右クリック
 - Run As > ant build を選択
 
 とすると、ANTの実行環境をインストールすることなくFat Jarを生成できるようになります。

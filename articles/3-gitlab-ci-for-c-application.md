---
title: "GitLab CI/CDを導入してC#/WPFアプリケーションのテストとインストーラーのビルド・デプロイを自動化する"
emoji: "🦊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["csharp", "dotnet", "wpf", "gitlab", "cicd"]
published: true
publication_name: "hacarus_blog"
---

# 概要
株式会社HACARUSプロダクト事業部の宇佐見です。HACARUSでは[HACARUS Check](https://check.hacarus.com/ja/)というプロダクトを開発しています。このプロダクトのアプリケーションの大半はC#にて開発されています。  
今回は、このアプリケーションのunit testの実行、及びビルド済みのアプリケーションの配布を自動化するために、GitLab CI/CDを導入した話をしたいと思います。

# GitLab CI/CDとは
GitLab CI/CDはGitLabが提供するCI/CDを実現するためのツールです。GitHubでいえば、GitHub Actionsのようなものです。  
HACARUSはGitのホスティングサービスとしてGitLabを利用しているため、GitLab CI/CDを利用することにしました。  
GitLab CI/CDは、`.gitlab-ci.yml`というファイルにCI/CDの設定を記述します。このファイルをリポジトリに追加することで、GitLab CI/CDが自動的にCI/CDを実行してくれます。例としてはこんな感じです。

```yaml
before_script:
  - "& 'C:/Program Files/Microsoft Visual Studio/2022/Professional/Common7/Tools/VsDevCmd.bat'"
  - chcp 65001

cache:
  paths:
    - ${CI_PROJECT_DIR}/HacarusCheckRobo/bin/x64/Release/net6.0-windows

stages:
  - pages
  - coverage
  - build-installer
  - changelog
  - release

include:
  - local: /.gitlab/test.gitlab-ci.yml
  - local: /.gitlab/coverage.gitlab-ci.yml
  - local: /.gitlab/build_installer.gitlab-ci.yml
  - local: /.gitlab/changelog.gitlab-ci.yml
  - local: /.gitlab/release.gitlab-ci.yml

```

上の例は実際に使っているgitlab-ci.ymlです。  
複数のファイルをincludeしていますが、これはCI/CDの設定を分割しているためです。stagesには、CI/CDの実行順序を記述します。上の例では、pages→coverage→build-installer→changelog→releaseの順で実行されます。  
ざっくりと各ステージがやっていることを書くと、
- pages: unittestの実行、及びcoverage reportを作成してGitLab Pagesにデプロイする
- coverage: coverageを計測して、リポジトリのバッジに反映する
- build-installer: インストーラーの作成
- changelog: リリースノートの作成
- release: GitLab Releaseにリリースを作成、GoogleDriveにビルド済みのインストーラーをアップロード

という様なことを行っています。

# テストを実行するジョブの例

下記がアプリケーションをビルドしてテストを実行するジョブの例です。
```yaml
pages:
  tags:
    - windows
  stage: pages
  script:
    - "& 'C:/Program Files (x86)/NuGet/nuget.exe' restore ./HacarusCheckRobo.sln"
    - "& 'C:/Program Files/Microsoft Visual Studio/2022/Professional/MSBuild/Current/Bin/MSBuild.exe' ./HacarusCheckRobo.sln -restore"
    - "& 'C:/Program Files/Microsoft Visual Studio/2022/Professional/MSBuild/Current/Bin/MSBuild.exe' ./HacarusCheckRobo.sln -t:Rebuild -p:Configuration=Release"
    - "& 'C:/Program Files/Microsoft Visual Studio/2022/Professional/Common7/IDE/CommonExtensions/Microsoft/TestWindow/vstest.console.exe' ./ModelTests/bin/x64/Release/net6.0-windows/ModelTests.dll"
    - "& 'C:/Program Files/Microsoft Visual Studio/2022/Professional/Common7/IDE/CommonExtensions/Microsoft/TestWindow/vstest.console.exe' ./HacarusCheckRoboTests/bin/x64/Release/net6.0-windows/HacarusCheckBasicTests.dll"
    - "& dotnet test ./HacarusCheckRobo.sln --collect:'XPlat Code Coverage' --results-directory coverageResult"
    - "& 'C:/Users/hacarus/.dotnet/tools/reportgenerator.exe' -reports:'./coverageResult/*/coverage.cobertura.xml' -targetdir:'coveragereport' -reporttypes:Html"
    - mv coveragereport public
  artifacts:
    paths:
    - public

```

初めに`nuget restore`を実行しています。`nuget restore`を実行しないと、`MSBuild.exe`が参照するパッケージが見つからないためです。  
また、最後にcoverage reportを作成しています。coverage reportは、GitLab Pagesにデプロイしています。  
GitLab Pagesは、GitLabが提供する静的サイトホスティングサービスです。coverage reportをGitLab Pagesにデプロイすることで、coverage reportをブラウザ上で確認できるようになります。


# GitLab CI/CDをどこで動かすか
一番シンプルなやり方はSaaSで提供されているGitLab CI/CDを利用することです。しかし、今回はアプリケーションをWindowsPC上でビルドする必要があるため、WindowsPC上でGitLab CI/CDを動かすことにしました。  
GitLab CI/CDをオンプレミスで動かす場合、GitLab Runnerというツールを利用します。GitLab Runnerは、GitLab CI/CDのジョブを実行するためのツールです。GitLab Runnerをインストールしたのち、Runnerをserviceとして登録すると、GitLab側のCI/CDの設定に従って、Runnerがジョブを実行してくれます。  

GitLabのUI上からRunnerを登録することができます。
![Runner](https://storage.googleapis.com/zenn-user-upload/43430d03378a-20230705.png)


# インストーラーのビルドについて
インストーラーのビルドには、[Wix Toolset](https://wixtoolset.org/)を利用しています。ビルド自体が7分ほどかかり、毎回ビルドするのはCI/CDの実行時間が長くなってしまうため、タグ付けされたときのみビルドするようにしています。  
タグ付けされたときのみビルドするためには、GitLab CI/CDの `only` keywordを利用し、tagsを指定します。例えば、以下のようになります。

```yaml
build-installer:
  stage: build-installer
  only:
    - tags
``` 
こうしてやると、タグづけされた時のみ、build-installerステージが有効になります。

# インストーラーのデプロイについて
インストーラーのデプロイ先には、GoogleDriveを利用しています。S3等も候補には上がったのですが、インストーラーをお客様に提供する必要があるので、セールスの人たちもアクセスしやすいサービスである必要があることからGoogleDriveになりました。  
GitLab CI/CDからGoogleDriveにアップロードするためには、[Google Drive API](https://developers.google.com/drive/api/v3/about-sdk)を利用する必要があります。そのまま使うのは辛かったので、[Google Drive APIをラップしたライブラリ](https://github.com/prasmussen/gdrive)を利用しています。ただし少し前にアーカイブされてしまったので、今後採用される方は新しい[こちら](https://github.com/glotlabs/gdrive)を利用することをおすすめします。
Google Drive APIの認証を行う方法は色々ありますが、[サービスアカウント](https://cloud.google.com/iam/docs/service-account-overview?hl=ja)を作成して、サービスアカウントにインストーラーをアップロードするフォルダのアクセス権を与えることで対応しました。

#  Releaseの作成について
GitLabが提供している[GitLab Release CLI tool](https://docs.gitlab.com/ee/user/project/releases/release_cli.html)を用いてReleaseを作成します。
以下は、GitLab Release CLI toolを利用してReleaseを作成するジョブの例です。

```yaml
release:
  image: registry.gitlab.com/gitlab-org/release-cli:latest
  stage: release
  needs: ["build-installer", "changelog"]
  before_script:
    - echo start release
  script:
    - CHANGELOG="$(cat changelog.md)"
    - release-cli create --name "Release $CI_COMMIT_TAG" --tag-name $CI_COMMIT_TAG --description="$CHANGELOG" --ref "$CI_COMMIT_SHA" --assets-link "{\"name\":\"Release Folder\",\"url\":\"https://drive.google.com/drive/u/0/folders/$PARENT_DIR_ID\"}"
  only:
    - tags

```

途中で`changelog.md`を参照していますが、これは事前のchangelogを作成するステージで作成されるマークダウンファイルになります。このファイルは[このReleaseに含まれるMRの情報を自動的に取得する仕組み](https://zenn.dev/hacarus_blog/articles/01aac587710b9d)を使って生成されます。  
また、release-cliに渡すURLの情報はインストーラーがデプロイされるGoogleDriveのフォルダへのリンクになっています。  
したがって、Release情報を見ることで、Releaseに含まれる更新情報及びそれに対応するインストーラーがどれかわかるようになっています。


# CI/CDを設定してよかったこと
- ユニットテストの実行が自動化された
   - 言うまでもないことですが、このおかげで過去の実装を破壊するような変更をしたときに、すぐに気づくことができました。
- 誰でもインストーラーが簡単にビルドできるようになった
   - インストーラーをビルドし、デプロイする手段を自動化することで、デプロイ時に複雑なコマンドを打つ必要がなくなりました。タグ付けさえしてしまえば誰でもボタン一つでインストーラーをビルドできるので、デプロイの手間が大幅に減りました。
- すぐに生成物をセールスの人に共有できるようになった
  - 以前は、インストーラーをビルドしたら、セールスの人に特定のフォルダに手動でアップロードしてお知らせするという手段をとっていました。CDの仕組みを導入したことで、タグづけしたら自動的にインストーラーがアップロードされるようになり、セールスの人に連絡するだけでよくなり成果物の共有が早くなりました。
- Release生成を自動化することで、Releaseに紐づく情報の管理が楽になった
  - Releaseに含まれる変更及びそれに対応するインストーラーがどれかがわかるようになったため、あるインストーラーで追加された機能などが一覧してわかるようになりました。
# まとめ
CI/CDの仕組みを導入することで、ユニットテストの自動化、インストーラー生成の自動化などを達成し、開発者がより開発に集中できるようになりました。  
ボタン一つで生成できるようになることで、簡単に誰でもできるようになるだけでなく、複雑な手順を他のメンバーに伝える必要がなくなったことも利点です。  
まだ手をつけられていないところ（RunnerをDockerコンテナ上で動かすなど）はあるので、今後も継続的に改善していきたいと思います。
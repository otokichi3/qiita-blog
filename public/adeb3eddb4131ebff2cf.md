---
title: PHPの自動テストフレームワークってどんなんがあるん？
tags:
  - PHP
  - テスト
private: false
updated_at: '2019-02-13T00:37:32+09:00'
id: adeb3eddb4131ebff2cf
organization_url_name: null
slide: false
ignorePublish: false
---
明日、勉強会で**自動テスト**を学ぶので、最低限の知識だけでも詰めておこうとしておく筆者の調べをアウトプットします。
筆者はこの記事を書いて早速 Testing Framework に多大な興味を寄せていますので、
皆様も興味を持っていただければと思います。

最初に、日本語のページを紹介しようと思いましたが、それでは単にまとめのコピーになるので、英語のページを参考にします。
[こちら](https://www.hongkiat.com/blog/automated-php-test/)を参考に、自動テストツールを挙げます。
※2018/10/27 の記事です。

## PHPUnit
> PHPUnit is the best-known testing framework for writing Unit Tests for PHP apps. 

PHP アプリの単体テストを記述するのに最もよく知られたテストフレームワーク。
言わずもがな、恐らく最も well-known な単体テストフレームワークではなかろうか。

## Codeception
> Codeception doesn’t only enable us to write Unit Tests, but also Functional and Acceptance Tests

信じられますか？
単体テストだけでなく、機能テストと受け入れテストも行えるようにしてくれる、ようです。

> It’s integrated with many PHP development frameworks such as Symfony2, Laravel4, Yii, Phalcon, and the Zend Framework

Symfony, Laravel, Yii, Phalcon, Zend Framework に入ってるみたいですね。ほう。

## Behat
> Behat is a popular behaviour-driven PHP testing framework

振舞駆動のテストフレームワーク。（ポピュラーかどうかは知らん）

> The tests we can write with Behat look rather like stories than code.

コードではなく物語を書く。
かっこいい。Cucumber に近いのか？

> The framework was inspired by the Cucumber project that’s a testing framework for the Ruby programming language.

Cucumber プロジェクトに触発されました。やっぱり。
Ruby: Cucumber なら、PHP: Behat としたいか。Ruby -> PHP のユーザに Cucumber -> Behat とする受け皿か。

自然な文で書けるということは、どういうテストをしたかが分かりやすいということ。
ただ、自然な英語なので導入できる環境に限りがある点が難点かもしれません。
（著者の同僚は TOEIC スコアに換算すると 300 点台ばかりとなる見込みです）

## PHPSpec
> PHPSpec also follows the behaviour-driven testing approach, but its other subtype called SpecBDD. 
> With PHPSpec we need to write the specifications first that describe how the application code will behave. 

振舞駆動のテストだが、SpecBDD という派生型のものだ！
これを使うと、コードがどのように動くかの仕様をまず書く必要がある、そう。
つまり仕様がないコードはなくなる、ということだ。え、凄くね？
形だけの仕様書を納品する SE 業界に変化を起こしてくれそう。
というか、それはすでに起きてるが著者が追いついていないんだろうことは明白。

> It was also inspired by a Ruby testing framework called RSpec.

なるほど RSpec に触発されたのか。Cucumber と RSpec は競い合う関係か。

## Storyplayer
> Storyplayer is a full-stack testing framework that makes it possible to write end-to-end tests for an entire platform. 

名前に恥じぬ強気な説明だ。
プラットフォーム全体の端から端までのテストを書くことが出来るようになるフルスタックテストフレームワークです。
つまり Storyplayer 以外何も要らない、と。なるほど守備範囲が広くて面白い。

## Peridot
> Peridot is a lightweight, extensible testing framework for PHP.

軽量で拡張できるテストフレームワークですよ。

紹介の仕方がうまい。まるで Peridot の手先のよう。

>  It features an event-driven architecture that allows testers to easily customize the framework via plugins and reporters.

プラグインやレポータにより、テスターが容易にフレームワークをカスタマイズできるイベント駆動アーキテクチャを備えています。
ううむ分からんが、拡張性があなたの自由度です、というところか。CodeIgniter ぐらいの立ち位置だろうか。

## Atoum
> Atoum is an intuitive and modern PHP testing framework that allows us to run unit tests.

単体テストを行える直感的でモダンな PHP テストフレーワームです。

> It simplifies test development, and as it’s a young framework it makes use of some newer capabilities that were introduced in PHP 5.3 (it can‘t be used with older PHP versions) to provide us with a quick and easy-to-understand testing process.

だいぶ省略するが、シンプルなテスト開発で簡単にテストの工程を理解できるよ、というところか。

> Atoum ensures a high level of security during test execution, as it isolates each test method in its own PHP process.

PHP とは別プロセスで動きます。へえ。

## Kahlan
カーラン、らしい。コーランではない。宗教色なくてよかった。

> Kahlan is a full-featured BDD testing framework that makes it possible to write Unit Tests using the describe-it syntax.

構文で単体テストを書ける BDD(behavior driven development; 振舞駆動開発) テストフレームワーク。

>  It embraces the KISS (Keep It Simple, Stupid) design principle. Kahlan requires at least PHP 5.5.

KISS の原則を取り入れている。最低でも PHP 5.5 だぞ。

> It has a small code base, it’s said to be about 10 times smaller than PHPUnit, and it has loads of features that provide us with an extensible and customizable testing workflow.

PHPUnit より 10 倍小さいコード。拡張可能でカスタマイズ可能なテスト処理。

## Selenium
来たよ本命。

> Selenium is a sophisticated testing framework that automates browsers. This means it’s possible to write User Acceptance Tests that examine the entire app as a whole.

洗練されたブラウザ自動化のテストフレームワーク。
アプリ全体のユーザ受け入れテストが可能、ということを意味する。

> Selenium is a robust tool that has its own WebDriver API that can drive a browser natively as though a real user would use it either locally or on a remote machine. Selenium is an excellent tool for testing more mature web applications.

とにかく優れているのは Selenium なんだ、と。ローカルでもリモートでも使えるし。

# まとめ
> There’s an important thing though that we always need to keep in mind. Automated testing may be powerful, but it can never substitute beta testing – tests done by real humans who will be the future users of the application.

肝に銘じておくべきことは、自動化されたテストが本当のユーザによって実施されるテストに置き換わるものではない、っていうことさ。

この比較順といい、比較対象といい、まとめといい、なんと良い記事なんと良い書き手だ。敬意を表す。
己、己の環境に相応しいテストツールを見極めてください。
私も見極めて、使ってみてこの英文記事に負け劣る記事を書きます、いずれ。

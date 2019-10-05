# Hitch

## 概要
犬による犬のための出会い系アプリ。

## 仕様書・設計書
仕様書・設計書は`/docs`配下に配置.

## 構成
Rails6 ベースの標準的なWebアプリケーション構成(非SPA)。JavaScript、CSS(SCSS)、画像などの静的コンテンツは全てWebpackerで管理。

## 各環境での動作方法

### ローカル環境
あとで書く。

### DEV環境
あとで書く。

### STG環境
あとで書く。

### 本番環境
あとで書く。

## コーディング規約

### Controller
- パラメーターの入力チェックをControllerで行わない。
  - パラメーターの入力チェックはModelのバリデーション機能を利用する。
  - テーブルに依存しないパラメーターの入力チェックは、Controller内に`ActiveModel::Model`をincludeしたサブクラスを作成し、そのクラスのインスタンスで行う。
``` ruby
class UserController < ApplicationController

  def create

    # ActiveModel::Modelをincludeしたクラスのインスタンスで入力チェック。
    request = CreateRequest.new(params_for_create)
    return render template: new if request.invlid?

    # 後続処理(省略)。

  end

  private

  def params_for_create
    params.require(:user).permit(:name)
  end

  class CreateRequest
    include ActiveModel::Model
    validates :name, presence: true
  end

end
```
- ビジネスロジックをControllerに記述しない。
  - ビジネスロジックはModelに記述する。
  - ただし複数のビジネスロジックに跨るトランザクションを貼る必要がある場合、トランザクション自体はControllerで作成・操作する。

### Model
あとで書く。

### View
- ビジネスロジックをViewに記述しない。
  - ビジネスロジックはModelに記述する。
- 共通テンプレート内でインスタンス変数を参照しない。
  - 共通テンプレート内でインスタンス変数を参照すると、テンプレートを呼び出す親ViewやController側で、そのためだけのインスタンス変数定義が必要になるため。
  - 共通テンプレートで利用したい動的な値は、共通テンプレート呼び出し時のローカル変数として渡す。

### Helper
あとで書く。

### JavaScript
- システム全体に共通する処理は下記のJavaScriptファイルに記述する。
  - `app/wp_assets/src/javascripts/global.js`
  - ただしグローバル関数、グローバル変数/定数は定義しない。
- 特定の画面でのみ行う処理は下記ディレクトリにコントローラー名と同じ名前のファイルを配置して、記述する。
  - `app/wp_assets/src/javascripts/#{コントローラー名}.js`
  - 上記のJavaScriptファイルではオブジェクトを`export default`する。
  - オブジェクトの中にはアクション名と同じキー名で、ページ表示時(厳密にはビルドされたJavaScriptファイル実行時)に実行したい関数オブジェクトを定義する。
  - 定義した関数オブジェクトはDOM構築完了よりも前に実行される。そのためDOMの構築完了を待ちたい場合は、`window.addEventListener('load', () => {})`でイベントをバインドする。
  - 定義した処理は、下記JavaScriptファイルに読み込み設定を記述する必要がある。
    - `app/wp_assets/src/javascripts/application.js`
    - 記述内容は上記のファイルを参照してください。
- ES2015以降の記法でコーディングを行う。
  - 具体的には変数宣言を`let`、定数宣言を`const`、関数をアロー関数式で定義するなど。
  - WebpackerがBableでトランスパイルを実施しているため、基本的にはIE11にも対応するJavaScriptがビルドされる。
- jQueryは利用しない。
  - BootstrapがjQueryに依存するため利用可能な状態にはなっているが、今後モダンなJSフレームワークを導入するような場合、jQueryに依存したソースコードは移植が困難になるので利用しない。

### CSS(SCSS)
- 厳密なコーディング規約は設けないが、OOCSS、BEM、SMACSSなどに代表される、コンポーネントを意識したコーディングを心がける。
- システム全体で利用する共通のスタイルは下記SCSSファイルに記述する。
  - `app/wp_assets/src/stylesheets/global.scss`
- システム全体で利用する共通の変数は下記SCSSファイルに記述する。
  - `app/wp_assets/src/stylesheets/_variables.scss`
- Bootstrapのスタイルを上書きする際は下記SCSSファイルに記述する。ただしシステム全体に影響しない、画面個別の上書きである場合は、後述するコントローラー単位のSCSSに記述する。
  - `app/wp_assets/src/stylesheets/bootstrap_override.scss`
- システム全体に影響しない、特定の画面に限定されるスタイルは下記のディレクトリにコントローラー単位のファイルを配置して、そこに記述する。
  - `app/wp_assets/src/stylesheets/コントローラー名.scss`
  - 上記ファイルはベースとなるSCSSファイルから`@import`で読み込む必要がある。ベースとなるSCSSは下記。
    - `app/wp_assets/src/stylesheets/application.scss`
  - 各画面のbodyタグにはコントローラー名と同じid、アクション名と同じclassが付与される。そのためコントローラー単位のスタイルは下記のように記述する。
``` scss
// コントローラー名がApplicationController、アクション名がindexだった場合。
#application {
  // コントローラー単位ではあるが画面単位ではないものはここに記述。
  &.index {
    // 画面単位であるものはここに記述。
  }
}
```
- 上記の通り、bodyタグに対して自動的に付与されるもの以外、原則としてIDセレクタは利用しない(=HTMLにもid属性を付与しない)。

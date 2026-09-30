# Rails Example - Definition of Done (DoD) and Estimation Details

## §4.2. 見積もりの詳細（Rails版）

### §4.2.1. ストーリポイント1に含まれるタスク構成（DoDベース - Rails）

スウォーム型Rails開発では、単に動くだけではなく「チームの資産」にするための作業を全て含めて1ポイントとします。

| 階層 | 具体的なタスク内容（1ポイントの範囲 - Rails） | スウォーム/知識共有のポイント |
|------|--------------------------------------|------------------------------|
| 設計 | RESTfulルート1つとDB列1つの定義 | Rails規約に沿った命名とルート設計を全員で確認する |
| DB | Migration作成 + カラム1つ（String型）を持つテーブル作成 + rollback確認 | `rails db:migrate`と`rails db:rollback`の手順を共有する |
| Model | ActiveRecord基本クラス作成 + 基本バリデーション1つ | Rails規約とActiveRecordパターンの書き方を統一する |
| Controller | 固定値を返すコントローラーアクション1つの実装 | Strong ParametersとRESTfulアクションの書き方を学ぶ |
| View | ERBテンプレート作成 + 取得した文字列を1つ表示するHTML | Rails Helpersとform_withの基本的な使い方を共有する |
| Route | config/routes.rbへのルート定義追加 | RESTful routingとnamed routeの命名規則を確認する |
| テスト | RSpec/Minitestでのモデル・コントローラー・統合テスト各1件 | FactoryBot/Fixturesの使い方とテスト実行手順を同期する |
| インフラ | Rails環境での起動確認 + 検証環境へのデプロイ | `rails server`起動とアセット管理の方法を全員が知る |
| ドキュメント | README.mdに新機能の起動方法とルート情報を追記 | Railsプロジェクトのドキュメント形式を統一する |

### §4.2.2. ストーリーポイントの見積もり基準表（Rails版）

1ポイントを基準とした場合の、Rails開発における他のポイントとの比較基準です。

| ポイント | 分類 | 判断基準（Rails開発） | スウォーム時の関わり方 |
|--------|------|----------|----------------------|
| 1 | 最小/定型 | Rails scaffoldレベル。基本的なCRUDの1操作で、ActiveRecordの標準機能のみ使用。 | Rails初学者のオンボーディングに最適。基本的なRails規約を学ぶ。 |
| 2 | 小/単純 | 関連テーブル1つとのアソシエーション。基本的なバリデーション2-3個。Simple form使用。 | 2名でペアプロし、残りはPRレビュー。ActiveRecordアソシエーションを習得。 |
| 3 | 中/標準 | 複数モデル間の関連処理。Serviceクラス導入。カスタムバリデーション。Ajax対応。 | 標準的なスウォーム単位。MVCの役割分担とRails wayを全員で議論。 |
| 5 | 大/複雑 | 外部gem導入（Devise, Sidekiq等）。API連携。複雑なクエリ最適化。ファイルアップロード機能。 | モブプロを推奨。Gemの選定理由とRailsアーキテクチャ設計を全員で固める。 |
| 8 | 巨大 | マイクロサービス分割。大幅なDB設計変更。新しいRailsバージョンへのアップグレード。 | PBI分割を検討。複数スプリントにまたがる可能性があるため再見積もりが必要。 |

### Rails開発における1ポイント（Hello World）の具体例

最もシンプルなRails開発における1ポイントの具体的な成果物は以下の通りです：

**例：「メッセージ一覧機能」1ポイント**

```ruby
# 1. Migration (db/migrate/xxx_create_messages.rb)
class CreateMessages < ActiveRecord::Migration[7.0]
  def change
    create_table :messages do |t|
      t.string :content
      t.timestamps
    end
  end
end

# 2. Model (app/models/message.rb)
class Message < ApplicationRecord
  validates :content, presence: true
end

# 3. Controller (app/controllers/messages_controller.rb)
class MessagesController < ApplicationController
  def index
    @messages = Message.all
  end
end

# 4. View (app/views/messages/index.html.erb)
<h1>Messages</h1>
<% @messages.each do |message| %>
  <p><%= message.content %></p>
<% end %>

# 5. Route (config/routes.rb)
Rails.application.routes.draw do
  resources :messages, only: [:index]
end

# 6. Test (spec/models/message_spec.rb)
RSpec.describe Message, type: :model do
  it "is valid with content" do
    message = Message.new(content: "Hello")
    expect(message).to be_valid
  end
end
```

この一連の実装により、`http://localhost:3000/messages`でメッセージ一覧が表示され、データベースからの取得・表示が確認できる状態になります。これがRails開発における「1ポイント」の基準となります。

### Rails特有の見積もり考慮要素

Rails開発では、以下の要素が見積もりに影響することを考慮してください：

- **Rails規約の習熟度**: Rails wayに沿った開発ができるかどうか
- **ActiveRecord の複雑さ**: アソシエーション、スコープ、コールバックの使用
- **Gem の依存関係**: 新しいGemの導入や既存Gemとの競合
- **アセット管理**: JavaScript、CSS、画像の管理方法（Sprockets/Webpacker）
- **Rails バージョン**: 使用するRailsバージョンによる機能差異
- **テスト環境**: RSpec vs Minitest、FactoryBot vs Fixtures の選択

これらの要素を踏まえて、チーム全体のRailsスキルレベルに応じた適切な見積もりを行うことが重要です。

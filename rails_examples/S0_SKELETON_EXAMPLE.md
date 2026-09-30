# S0 スケルトンチェックリスト Example (Ruby on Rails + RSpec)

本ドキュメントは [README.md](../README.md) §3.4.3「S0 スケルトンチェックリスト」の Rails 特化例です。Rails + RSpec + FactoryBot + Capybara（System Spec）構成での S0 完了時の成果物一覧を示します。汎用項目との対応は §3.4.3 本文を参照してください。

> **注記（視点について）**: 本ドキュメントは、人間が主体となって実装する Phase 1（README.md §1.4）を前提とした具体例です。バディ AI と分担する Phase 2、自律 AI と並走する Phase 3 では、スケルトン雛形の生成主体やレビューの粒度が変化しますが、S0 で揃えるべき成果物の集合自体は不変です。守るべきは「Sn より先（S1/S2）で追加コードを書き始めるための最小骨格を、検証可能な単位で揃える」ことであり、コード量や所要時間の絶対値ではありません。

**トラックA（機能実装）**

- [ ] Migration ファイル（空の、または最小限のスキーマ）: `db/migrate/YYYYMMDDHHMMSS_create_xxx.rb`
- [ ] Model スケルトン: `app/models/xxx.rb`（空のclass定義、必要最小限の `validates`）
- [ ] Controller スケルトン: `app/controllers/xxx_controller.rb`（空のaction、または `head :not_implemented`）
- [ ] Routes: `config/routes.rb` にエンドポイント定義を追加
- [ ] View / Serializer スケルトン: `app/views/xxx/` または `app/serializers/xxx_serializer.rb`
- [ ] 最初のユニットテスト（failする1本）: `spec/models/xxx_spec.rb`

**トラックB（受け入れテスト）**

- [ ] System Spec のラッパースクリプト（pending）: `spec/system/xxx_spec.rb` で `pending` または `skip`
- [ ] FactoryBot の Factory 下書き: `spec/factories/xxx.rb`

**このProject固有の確認事項**

| 項目 | 確認ポイント |
| --- | --- |
| テストフレームワーク | RSpec（`spec/` 配下） |
| Factory | FactoryBot（`spec/factories/`） |
| E2E | Capybara + System Spec（`spec/system/`） |
| Service層 | `app/services/` の有無を確認して必要なら追加 |
| Form Object | `app/forms/` の有無を確認 |
| Migration命名 | `create_xxx` / `add_xxx_to_yyy` の規約に従う |

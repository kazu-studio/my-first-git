---
name: diagram-ops
description: IT運用手順書を「配属されたばかりの新人」向けに、すごろく型の視覚的フロー図解として生成するスキル。「手順を図解して」「フロー図を作って」「運用手順を可視化して」「新人向けに図解して」「すごろく図を作って」と依頼された際に使用する。Tailwind CSS + Lucide iconでHTMLを生成し、surge.shにデプロイする。
---

# Diagram Ops

IT運用手順を**配属されたばかりの新人**が「今自分が全体のどこにいて、次にどのツールを使って何を確認すべきか」を直感的に把握できる、すごろく型フロー図解に変換する。

## 対象読者の前提

- 業務フローを初めて見る新人
- ミスが許されない環境で慎重に作業する必要がある
- テキスト手順書だけでは全体像が掴みづらい

## ゴール

図解を見た後、新人が「自分が今全体の何ステップ目にいて、次にどのツールで何を確認すれば良いか」を自分で判断できる状態にする。

---

## ワークフロー

### Phase 1: 手順の構造化

入力情報（テキスト、ドキュメント等）を [references/step-elements.md](references/step-elements.md) のテンプレートに従って整理する。

- 新人が迷う用語の補足、ツール名の具体化、注意レベルの判定をこの段階で完了させる。

### Phase 2: HTML生成

[references/visual-pattern.md](references/visual-pattern.md) のデザイン定義とHTML構造を完全に遵守してコードを生成する。

- Lucideアイコンの選定、Tailwindクラスの適用、JSによる進捗管理機能を正確に実装すること。

### Phase 3: 批判的レビュー

[references/review-prompt.md](references/review-prompt.md) をシステムプロンプトとして、サブエージェント（generalPurpose）にHTMLを読み込ませてレビューを実行する。

### Phase 4: ブラッシュアップ

レビュー結果および [references/exemplar.md](references/exemplar.md) の品質チェックリストに基づき、HTMLを最終修正する。

### Phase 5: ファイル保存

完成したHTMLファイルを `output/` ディレクトリに保存する。ファイル名は「手順名（ケバブケース）.html」とする。

### Phase 6: デプロイ

surge.sh を使用してデプロイを実行する。ドメイン名は「手順名（ケバブケース）.surge.sh」とする。

### Phase 7: ブラウザで開く

デプロイ完了後、`open` コマンドを使用してデプロイ先URLをブラウザで開く。

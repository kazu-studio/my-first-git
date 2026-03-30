# HTML構造ガイド（すごろく型フロー図解）

## デザイン原則（厳守事項）

1. **技術スタック**: Tailwind CSS (CDN) + Lucide Icon (CDN)
2. **カラー定義**:
   - 通常: `step-normal` (Blue)
   - 注意: `step-caution` (Amber)
   - 危険: `step-danger` (Red)
3. **アイコン**: 絵文字は一切禁止。必ず Lucide Icon を使用すること。
4. **ヘッダー**: ダークネイビー系のグラデーション（slate-800 to 600）
5. **進捗管理**: 「今ここ」ボタンとLocalStorageによる状態保持を必須とする。

---

## 基本テンプレート

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>【手順名】- 運用フロー図解</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/lucide@latest"></script>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700;800;900&family=Noto+Sans+JP:wght@400;500;700&display=swap" rel="stylesheet">
  <style>
    body { font-family: 'Inter', 'Noto Sans JP', sans-serif; background-color: #f8fafc; color: #1e293b; }

    /* ステップカード：白ベース＋左の太線アクセントに変更 */
    .step-card {
      background: white;
      border: 1px solid #e2e8f0;
      border-left-width: 6px;
      transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    }

    /* 注意レベル別の線色：PDFの視認性を継承 */
    .step-normal  { border-left-color: #3b82f6; } /* 青 */
    .step-caution { border-left-color: #f59e0b; } /* 橙 */
    .step-danger  { border-left-color: #ef4444; } /* 赤 */

    /* 現在地の強調スタイル。背景反転を廃止し、浮き出しと外枠で表現 */
    .step-card.step-current {
      border-left-width: 8px;
      box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.1);
      transform: translateY(-2px);
      outline: 3px solid #3b82f6;
      outline-offset: 2px;
    }

    /* パルスアニメーション付きの「ACTIVE」バッジ */
    @keyframes pulse-soft {
      0%, 100% { opacity: 1; transform: scale(1); }
      50% { opacity: 0.8; transform: scale(1.05); }
    }
    .now-badge {
      background: #3b82f6;
      color: white;
      animation: pulse-soft 2s infinite;
      font-size: 0.65rem;
      font-weight: 900;
      padding: 2px 8px;
      border-radius: 4px;
    }

    /* 接続矢印 */
    .arrow-down  { color: #94a3b8; }
    .arrow-right { color: #94a3b8; }

    /* 分岐カード */
    .branch-card {
      background: #fefce8;
      border: 2px dashed #eab308;
      border-radius: 0.75rem;
      padding: 1.25rem;
    }

    /* 進捗バー */
    .progress-bar {
      height: 8px;
      background: #e2e8f0;
      border-radius: 9999px;
      overflow: hidden;
    }
    .progress-fill {
      height: 100%;
      background: linear-gradient(90deg, #2563eb, #60a5fa);
      border-radius: 9999px;
      transition: width 0.4s ease;
    }

    /* Phase見出し */
    .phase-header {
      display: flex;
      align-items: center;
      gap: 12px;
      margin: 48px 0 24px 0;
    }
    .phase-badge {
      background: #1e293b;
      color: white;
      padding: 4px 12px;
      border-radius: 8px;
      font-size: 0.75rem;
      font-weight: 800;
      letter-spacing: 0.05em;
    }
  </style>
</head>
<body class="bg-slate-50">

  <!-- ヘッダー -->
  <header class="bg-gradient-to-r from-slate-800 to-slate-600 text-white py-8 px-4">
    <div class="max-w-3xl mx-auto">
      <div class="flex items-center gap-3 mb-2">
        <i data-lucide="clipboard-list" class="w-8 h-8 text-blue-300"></i>
        <h1 class="text-2xl md:text-3xl font-bold">【手順名】</h1>
      </div>
      <p class="text-slate-300 text-sm">全 {N} ステップ ／ 対象ツール: {ツール名}</p>
    </div>
  </header>

  <!-- 進捗トラッカー -->
  <div class="sticky top-0 z-40 bg-white border-b border-slate-200 shadow-sm">
    <div class="max-w-3xl mx-auto px-4 py-3 flex items-center gap-4">
      <span class="text-sm font-medium text-slate-600 whitespace-nowrap">進捗</span>
      <div class="progress-bar flex-1">
        <div class="progress-fill" id="progressFill" style="width: 0%"></div>
      </div>
      <span class="text-sm font-bold text-blue-700" id="progressText">0 / {N}</span>
    </div>
  </div>

  <!-- メインフロー -->
  <main class="max-w-3xl mx-auto px-4 py-8">

    <!-- ===== 各ステップカード ===== -->
    <!-- step-normal / step-caution / step-danger のいずれかを付与 -->
    <div class="step-card step-normal rounded-2xl p-6 mb-4 shadow-sm" id="step-1">
      <div class="flex justify-between items-start mb-4">
        <div class="flex items-center gap-3">
          <span class="w-8 h-8 bg-blue-100 text-blue-600 rounded-lg flex items-center justify-center font-bold">1</span>
          <h3 class="text-lg font-bold text-slate-800">【アクション名】</h3>
        </div>
        <button onclick="markCurrent(1)" class="text-xs font-bold text-blue-600 hover:bg-blue-50 px-3 py-1.5 rounded-lg border border-blue-200 transition-colors now-btn">今ここ</button>
      </div>

      <div class="space-y-3">
        <!-- Location / Tool -->
        <div class="flex items-start gap-3 bg-slate-50 p-3 rounded-xl border border-slate-100">
          <i data-lucide="map-pin" class="w-5 h-5 text-slate-400 mt-0.5"></i>
          <div>
            <div class="text-[10px] font-bold text-slate-400 uppercase">Location / Tool</div>
            <p class="text-sm font-medium">【場所や使用ツールを記載】</p>
          </div>
        </div>
        <!-- Action -->
        <div class="flex items-start gap-3 bg-blue-50/50 p-3 rounded-xl border border-blue-100">
          <i data-lucide="mouse-pointer-2" class="w-5 h-5 text-blue-500 mt-0.5"></i>
          <div>
            <div class="text-[10px] font-bold text-blue-400 uppercase">Action</div>
            <p class="text-sm">【具体的な操作手順を記載】</p>
          </div>
        </div>
        <!-- Check -->
        <div class="flex items-start gap-3 bg-emerald-50/50 p-3 rounded-xl border border-emerald-100">
          <i data-lucide="check-circle-2" class="w-5 h-5 text-emerald-500 mt-0.5"></i>
          <div>
            <div class="text-[10px] font-bold text-emerald-400 uppercase">Check</div>
            <p class="text-sm font-medium text-emerald-900">【〜であることを確認】</p>
          </div>
        </div>
      </div>
    </div>

    <!-- 接続矢印（通常） -->
    <div class="flex justify-center my-1">
      <i data-lucide="arrow-down" class="w-6 h-6 arrow-down"></i>
    </div>

    <!-- ===== 分岐ステップの例 ===== -->
    <div class="step-card step-caution rounded-2xl p-6 mb-4 shadow-sm" id="step-2">
      <div class="flex justify-between items-start mb-4">
        <div class="flex items-center gap-3">
          <span class="w-8 h-8 bg-amber-100 text-amber-600 rounded-lg flex items-center justify-center font-bold">2</span>
          <h3 class="text-lg font-bold text-slate-800">【分岐アクション名】</h3>
        </div>
        <button onclick="markCurrent(2)" class="text-xs font-bold text-amber-600 hover:bg-amber-50 px-3 py-1.5 rounded-lg border border-amber-200 transition-colors now-btn">今ここ</button>
      </div>

      <div class="space-y-3">
        <div class="flex items-start gap-3 bg-slate-50 p-3 rounded-xl border border-slate-100">
          <i data-lucide="map-pin" class="w-5 h-5 text-slate-400 mt-0.5"></i>
          <div>
            <div class="text-[10px] font-bold text-slate-400 uppercase">Location / Tool</div>
            <p class="text-sm font-medium">【場所や使用ツールを記載】</p>
          </div>
        </div>
        <div class="flex items-start gap-3 bg-blue-50/50 p-3 rounded-xl border border-blue-100">
          <i data-lucide="mouse-pointer-2" class="w-5 h-5 text-blue-500 mt-0.5"></i>
          <div>
            <div class="text-[10px] font-bold text-blue-400 uppercase">Action</div>
            <p class="text-sm">【具体的な操作手順を記載】</p>
          </div>
        </div>
        <div class="flex items-start gap-3 bg-emerald-50/50 p-3 rounded-xl border border-emerald-100">
          <i data-lucide="check-circle-2" class="w-5 h-5 text-emerald-500 mt-0.5"></i>
          <div>
            <div class="text-[10px] font-bold text-emerald-400 uppercase">Check</div>
            <p class="text-sm font-medium text-emerald-900">【〜であることを確認】</p>
          </div>
        </div>
        <!-- 注意ブロック（cautionステップに追加） -->
        <div class="flex items-start gap-3 bg-amber-50 p-3 rounded-xl border border-amber-200">
          <i data-lucide="triangle-alert" class="w-5 h-5 text-amber-600 mt-0.5"></i>
          <div>
            <div class="text-[10px] font-bold text-amber-500 uppercase">Caution</div>
            <p class="text-sm text-amber-800">【注意事項を記載】</p>
          </div>
        </div>
      </div>

      <!-- 分岐 -->
      <div class="grid grid-cols-2 gap-3 mt-4">
        <div class="branch-card text-center">
          <i data-lucide="check" class="w-5 h-5 text-green-600 mx-auto mb-1"></i>
          <p class="text-sm font-bold text-green-700">OK の場合</p>
          <p class="text-xs text-slate-600 mt-1">→ Step 3 へ</p>
        </div>
        <div class="branch-card text-center">
          <i data-lucide="x" class="w-5 h-5 text-red-600 mx-auto mb-1"></i>
          <p class="text-sm font-bold text-red-700">エラーの場合</p>
          <p class="text-xs text-slate-600 mt-1">→ エラー対応へ</p>
        </div>
      </div>
    </div>

    <!-- ===== 危険ステップの例 ===== -->
    <div class="step-card step-danger rounded-2xl p-6 mb-4 shadow-sm" id="step-3">
      <div class="flex justify-between items-start mb-4">
        <div class="flex items-center gap-3">
          <span class="w-8 h-8 bg-red-100 text-red-600 rounded-lg flex items-center justify-center font-bold">3</span>
          <h3 class="text-lg font-bold text-slate-800">【危険アクション名】</h3>
          <span class="text-xs bg-red-600 text-white px-2 py-0.5 rounded font-bold">要注意</span>
        </div>
        <button onclick="markCurrent(3)" class="text-xs font-bold text-red-600 hover:bg-red-50 px-3 py-1.5 rounded-lg border border-red-200 transition-colors now-btn">今ここ</button>
      </div>

      <div class="space-y-3">
        <div class="flex items-start gap-3 bg-slate-50 p-3 rounded-xl border border-slate-100">
          <i data-lucide="map-pin" class="w-5 h-5 text-slate-400 mt-0.5"></i>
          <div>
            <div class="text-[10px] font-bold text-slate-400 uppercase">Location / Tool</div>
            <p class="text-sm font-medium">【場所や使用ツールを記載】</p>
          </div>
        </div>
        <div class="flex items-start gap-3 bg-blue-50/50 p-3 rounded-xl border border-blue-100">
          <i data-lucide="mouse-pointer-2" class="w-5 h-5 text-blue-500 mt-0.5"></i>
          <div>
            <div class="text-[10px] font-bold text-blue-400 uppercase">Action</div>
            <p class="text-sm">【具体的な操作手順を記載】</p>
          </div>
        </div>
        <div class="flex items-start gap-3 bg-emerald-50/50 p-3 rounded-xl border border-emerald-100">
          <i data-lucide="check-circle-2" class="w-5 h-5 text-emerald-500 mt-0.5"></i>
          <div>
            <div class="text-[10px] font-bold text-emerald-400 uppercase">Check</div>
            <p class="text-sm font-medium text-emerald-900">【〜であることを確認】</p>
          </div>
        </div>
        <!-- 危険ブロック（dangerステップに追加） -->
        <div class="flex items-start gap-3 bg-red-50 p-3 rounded-xl border border-red-200">
          <i data-lucide="shield-alert" class="w-5 h-5 text-red-700 mt-0.5"></i>
          <div>
            <div class="text-[10px] font-bold text-red-500 uppercase">Danger</div>
            <p class="text-sm font-medium text-red-800">【取り返しのつかないミスにつながる注意事項】</p>
          </div>
        </div>
      </div>
    </div>

    <!-- 完了カード -->
    <div class="flex justify-center my-1">
      <i data-lucide="arrow-down" class="w-6 h-6 arrow-down"></i>
    </div>
    <div class="bg-green-50 border-2 border-green-400 rounded-xl p-6 text-center">
      <i data-lucide="check-circle-2" class="w-12 h-12 text-green-600 mx-auto mb-2"></i>
      <p class="font-bold text-xl text-green-800">作業完了</p>
      <p class="text-sm text-green-700 mt-1">【完了後の確認・報告先など】</p>
    </div>

  </main>

  <!-- Lucide初期化 + 現在位置JS -->
  <script>
    lucide.createIcons();

    const TOTAL_STEPS = {N};  // ステップ総数に差し替え
    let currentStep = parseInt(localStorage.getItem('ops-current-step') || '0');

    function markCurrent(stepNum) {
      currentStep = stepNum;
      localStorage.setItem('ops-current-step', stepNum);
      updateUI();
    }

    function updateUI() {
      // 全ステップカードをリセット
      document.querySelectorAll('.step-card').forEach(card => {
        card.classList.remove('step-current');
        const badge = card.querySelector('.now-badge');
        if (badge) badge.remove();
      });

      if (currentStep > 0) {
        const card = document.getElementById('step-' + currentStep);
        if (card) {
          card.classList.add('step-current');
          const header = card.querySelector('.flex.justify-between');
          const badge = document.createElement('span');
          badge.className = 'now-badge';
          badge.textContent = '▶ 今ここ';
          header.appendChild(badge);
          card.scrollIntoView({ behavior: 'smooth', block: 'center' });
        }
      }

      // 進捗バー更新
      const pct = currentStep > 0 ? Math.round((currentStep / TOTAL_STEPS) * 100) : 0;
      document.getElementById('progressFill').style.width = pct + '%';
      document.getElementById('progressText').textContent =
        currentStep > 0 ? currentStep + ' / ' + TOTAL_STEPS : '0 / ' + TOTAL_STEPS;
    }

    // ページ読み込み時に復元
    if (currentStep > 0) updateUI();
  </script>
</body>
</html>
```

---

## Phase見出し（大手順の区切り）

手順が複数のフェーズに分かれる場合、ステップ群の前に配置して「ステージの入口」を視覚的に示す。

```html
<div class="phase-header">
  <span class="phase-badge">PHASE 1</span>
  <h2 class="text-xl font-bold text-slate-800 tracking-tight">【フェーズ名】</h2>
  <div class="flex-1 h-px bg-slate-200"></div>
</div>
```

---

## カラー定義（注意レベル別）

| レベル | 用途 | クラス | 左ボーダー色 | バッジ背景 |
|--------|------|--------|-------------|-----------|
| 通常 | 問題なく実行できるステップ | `step-normal` | `#3b82f6`（青） | `bg-blue-100 text-blue-600` |
| 注意 | 確認が必要・分岐がある | `step-caution` | `#f59e0b`（橙） | `bg-amber-100 text-amber-600` |
| 危険 | ミスが取り返しのつかない結果を招く | `step-danger` | `#ef4444`（赤） | `bg-red-100 text-red-600` |
| 現在地 | 「今ここ」でマークされた状態 | `step-current`（JS付与） | 左8px＋outline | — |

---

## よく使うLucide Iconリスト（運用用途）

| 用途 | アイコン名 | コード |
|------|-----------|--------|
| 場所・ツール（Locationブロック） | `map-pin` | `<i data-lucide="map-pin" class="w-5 h-5 text-slate-400"></i>` |
| 操作（Actionブロック） | `mouse-pointer-2` | `<i data-lucide="mouse-pointer-2" class="w-5 h-5 text-blue-500"></i>` |
| 確認（Checkブロック） | `check-circle-2` | `<i data-lucide="check-circle-2" class="w-5 h-5 text-emerald-500"></i>` |
| 注意（Cautionブロック） | `triangle-alert` | `<i data-lucide="triangle-alert" class="w-5 h-5 text-amber-600"></i>` |
| 危険（Dangerブロック） | `shield-alert` | `<i data-lucide="shield-alert" class="w-5 h-5 text-red-700"></i>` |
| ターミナル | `terminal` | `<i data-lucide="terminal" class="w-4 h-4 text-slate-600"></i>` |
| データベース | `database` | `<i data-lucide="database" class="w-4 h-4 text-purple-500"></i>` |
| サーバー | `server` | `<i data-lucide="server" class="w-4 h-4 text-slate-600"></i>` |
| ログ・記録 | `file-text` | `<i data-lucide="file-text" class="w-4 h-4 text-blue-500"></i>` |
| 手順書 | `clipboard-list` | `<i data-lucide="clipboard-list" class="w-6 h-6 text-blue-300"></i>` |
| 完了カード | `check-circle-2` | `<i data-lucide="check-circle-2" class="w-12 h-12 text-green-600"></i>` |
| 分岐OK | `check` | `<i data-lucide="check" class="w-5 h-5 text-green-600"></i>` |
| 分岐NG | `x` | `<i data-lucide="x" class="w-5 h-5 text-red-600"></i>` |
| 矢印（下） | `arrow-down` | `<i data-lucide="arrow-down" class="w-6 h-6 text-slate-400"></i>` |

---

## 接続パターン

### 通常の順序フロー

```html
<!-- ステップN -->
<div class="step-card step-normal ...">...</div>

<!-- 矢印 -->
<div class="flex justify-center my-1">
  <i data-lucide="arrow-down" class="w-6 h-6 arrow-down"></i>
</div>

<!-- ステップN+1 -->
<div class="step-card step-normal ...">...</div>
```

### 分岐フロー（2方向）

```html
<!-- 分岐ステップ内 -->
<div class="grid grid-cols-2 gap-3 mt-3">
  <div class="branch-card text-center">
    <i data-lucide="check" class="w-5 h-5 text-green-600 mx-auto mb-1"></i>
    <p class="text-sm font-bold text-green-700">OK の場合</p>
    <p class="text-xs text-slate-600 mt-1">→ Step {N} へ</p>
  </div>
  <div class="branch-card text-center">
    <i data-lucide="x" class="w-5 h-5 text-red-600 mx-auto mb-1"></i>
    <p class="text-sm font-bold text-red-700">エラーの場合</p>
    <p class="text-xs text-slate-600 mt-1">→ {対応先} へ</p>
  </div>
</div>
```

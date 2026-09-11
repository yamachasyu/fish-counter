# fish-counter
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>魚購入＆カウント管理</title>
  <style>
    :root {
      --bg-color: #f0f4f8;
      --card-bg: #ffffff;
      --primary: #0077b6;
      --accent: #00b4d8;
      --danger: #e63946;
      --text: #1d3557;
      --sub-text: #6c757d;
    }

    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      background-color: var(--bg-color);
      color: var(--text);
      margin: 0;
      padding: 16px;
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    h1 {
      margin-bottom: 12px;
      font-size: 1.5rem;
    }

    /* 総額表示エリア */
    .total-box {
      width: 100%;
      max-width: 400px;
      background: var(--primary);
      color: white;
      padding: 16px;
      border-radius: 12px;
      text-align: center;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
      margin-bottom: 20px;
      box-sizing: border-box;
    }

    .total-label {
      font-size: 0.9rem;
      opacity: 0.9;
    }

    .total-amount {
      font-size: 2.2rem;
      font-weight: bold;
      margin-top: 4px;
    }

    /* 入力フォーム */
    .add-fish-box {
      width: 100%;
      max-width: 400px;
      background: var(--card-bg);
      padding: 16px;
      border-radius: 12px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.05);
      margin-bottom: 20px;
      display: flex;
      flex-direction: column;
      gap: 12px;
      box-sizing: border-box;
    }

    .form-group {
      display: flex;
      flex-direction: column;
      gap: 4px;
    }

    .form-group label {
      font-size: 0.85rem;
      color: var(--sub-text);
      font-weight: bold;
    }

    input[type="text"], input[type="number"], input[type="date"] {
      width: 100%;
      padding: 10px;
      font-size: 1rem;
      border: 1px solid #ccc;
      border-radius: 8px;
      box-sizing: border-box;
    }

    .form-row {
      display: flex;
      gap: 8px;
    }

    .form-row .form-group {
      flex: 1;
    }

    button.add-btn {
      width: 100%;
      padding: 12px;
      background-color: var(--primary);
      color: white;
      border: none;
      border-radius: 8px;
      font-size: 1rem;
      font-weight: bold;
      cursor: pointer;
      margin-top: 4px;
    }

    button.add-btn:hover {
      opacity: 0.9;
    }

    /* リストエリア */
    .counter-list {
      width: 100%;
      max-width: 400px;
      display: flex;
      flex-direction: column;
      gap: 12px;
    }

    .counter-card {
      background: var(--card-bg);
      border-radius: 12px;
      padding: 16px;
      box-shadow: 0 4px 6px rgba(0,0,0,0.05);
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .fish-info {
      display: flex;
      flex-direction: column;
      gap: 4px;
    }

    .fish-date {
      font-size: 0.8rem;
      color: var(--sub-text);
    }

    .fish-name {
      font-weight: bold;
      font-size: 1.2rem;
    }

    .fish-price {
      font-size: 0.95rem;
      color: var(--primary);
      font-weight: bold;
    }

    .count-section {
      display: flex;
      align-items: center;
      gap: 8px;
    }

    .fish-count {
      font-size: 1.5rem;
      font-weight: bold;
      min-width: 32px;
      text-align: center;
    }

    .btn-count {
      width: 44px;
      height: 44px;
      border-radius: 50%;
      border: none;
      font-size: 1.2rem;
      font-weight: bold;
      color: white;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .btn-plus {
      background-color: var(--accent);
    }

    .btn-minus {
      background-color: #adb5bd;
    }

    .btn-delete {
      background: none;
      border: none;
      color: var(--danger);
      font-size: 0.8rem;
      margin-top: 6px;
      cursor: pointer;
      padding: 0;
      text-align: left;
    }
  </style>
</head>
<body>

  <h1>🐟 魚カウンター＆購入管理</h1>

  <!-- 総額表示エリア -->
  <div class="total-box">
    <div class="total-label">総購入金額</div>
    <div class="total-amount" id="totalAmount">¥0</div>
  </div>

  <!-- 魚・購入情報追加フォーム -->
  <form class="add-fish-box" onsubmit="addFish(event)">
    <div class="form-group">
      <label for="fishDateInput">購入日</label>
      <input type="date" id="fishDateInput" required>
    </div>

    <div class="form-row">
      <div class="form-group">
        <label for="fishNameInput">魚種名</label>
        <input type="text" id="fishNameInput" placeholder="例: マグロ" required>
      </div>
      <div class="form-group">
        <label for="fishPriceInput">価格 (円)</label>
        <input type="number" id="fishPriceInput" placeholder="例: 500" min="0">
      </div>
    </div>

    <button type="submit" class="add-btn">追加する</button>
  </form>

  <!-- リスト表示エリア -->
  <div class="counter-list" id="counterList"></div>

  <script>
    // ローカルタイムで「YYYY-MM-DD」を設定（UTCズレ防止）
    const today = new Date();
    const localDate = today.getFullYear() + '-' + String(today.getMonth() + 1).padStart(2, '0') + '-' + String(today.getDate()).padStart(2, '0');
    document.getElementById('fishDateInput').value = localDate;

    // XSS対策用のテキストエスケープ処理
    function escapeHtml(str) {
      return str.replace(/[&<>'"]/g, function(tag) {
        return {
          '&': '&amp;',
          '<': '&lt;',
          '>': '&gt;',
          "'": '&#39;',
          '"': '&quot;'
        }[tag] || tag;
      });
    }

    // ローカルストレージからデータ取得
    let fishData = JSON.parse(localStorage.getItem('fishCounters')) || [];

    // 保存＆再描画
    function saveAndRender() {
      localStorage.setItem('fishCounters', JSON.stringify(fishData));
      render();
    }

    // 魚の追加（Form submitに対応）
    function addFish(e) {
      if (e) e.preventDefault();

      const dateInput = document.getElementById('fishDateInput');
      const nameInput = document.getElementById('fishNameInput');
      const priceInput = document.getElementById('fishPriceInput');

      const date = dateInput.value;
      const name = nameInput.value.trim();
      const price = parseInt(priceInput.value, 10) || 0;

      if (!name) {
        alert('魚種名を入力してください');
        nameInput.focus();
        return;
      }

      fishData.push({
        id: Date.now(),
        date: date || '日付なし',
        name: name,
        price: price,
        count: 1
      });

      // 入力欄のリセット
      nameInput.value = '';
      priceInput.value = '';
      
      saveAndRender();
    }

    // 匹数の変更
    function changeCount(id, delta) {
      const fish = fishData.find(item => item.id === id);
      if (fish) {
        fish.count = Math.max(0, fish.count + delta);
        saveAndRender();
      }
    }

    // データの削除
    function deleteFish(id) {
      if (confirm('この項目を削除しますか？')) {
        fishData = fishData.filter(item => item.id !== id);
        saveAndRender();
      }
    }

    // 画面の更新処理
    function render() {
      const list = document.getElementById('counterList');
      const totalAmountEl = document.getElementById('totalAmount');
      list.innerHTML = '';

      let grandTotal = 0;

      fishData.forEach(fish => {
        const itemTotal = fish.price * fish.count;
        grandTotal += itemTotal;

        const card = document.createElement('div');
        card.className = 'counter-card';
        card.innerHTML = `
          <div class="fish-info">
            <span class="fish-date">📅 ${escapeHtml(fish.date)}</span>
            <span class="fish-name">${escapeHtml(fish.name)}</span>
            <span class="fish-price">¥${fish.price.toLocaleString()} / 匹 (計: ¥${itemTotal.toLocaleString()})</span>
            <button type="button" class="btn-delete" onclick="deleteFish(${fish.id})">削除</button>
          </div>
          <div class="count-section">
            <button type="button" class="btn-count btn-minus" onclick="changeCount(${fish.id}, -1)">-</button>
            <span class="fish-count">${fish.count}</span>
            <button type="button" class="btn-count btn-plus" onclick="changeCount(${fish.id}, 1)">+</button>
          </div>
        `;
        list.appendChild(card);
      });

      totalAmountEl.textContent = `¥${grandTotal.toLocaleString()}`;
    }

    // 初回表示
    render();
  </script>
</body>
</html>
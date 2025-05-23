<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>交通費申請システム</title>
  <style>
    /* ===== style.css + admin.css の統合 ===== */
    body {
      font-family: 'Segoe UI', sans-serif;
      margin: 0;
      padding: 0;
      background-color: #f5f5f5;
    }
    .container {
      max-width: 800px;
      margin: 2rem auto;
      padding: 2rem;
      background: white;
      border-radius: 10px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    }
    .login-page {
      background: linear-gradient(to right, #6dd5ed, #2193b0);
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }
    .login-box {
      width: 100%;
      max-width: 400px;
      padding: 2rem;
      background: white;
      border-radius: 10px;
      box-shadow: 0 5px 15px rgba(0,0,0,0.1);
    }
    input, select, button {
      width: 100%;
      padding: 0.75rem;
      margin: 0.5rem 0 1rem;
      border: 1px solid #ddd;
      border-radius: 5px;
      font-size: 1rem;
    }
    button {
      background-color: #2193b0;
      color: white;
      border: none;
      cursor: pointer;
      transition: background 0.3s;
    }
    button:hover {
      background-color: #17627b;
    }
    #dataTable {
      width: 100%;
      border-collapse: collapse;
      margin-top: 1rem;
    }
    #dataTable th, #dataTable td {
      padding: 0.75rem;
      border: 1px solid #ddd;
      text-align: left;
    }
    #dataTable th {
      background-color: #f0f0f0;
    }
    header {
      background-color: #343a40;
      color: white;
      padding: 1rem;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    @media (max-width: 768px) {
      .container {
        margin: 1rem;
        padding: 1rem;
      }
      #dataTable {
        display: block;
        overflow-x: auto;
      }
    }
  </style>
</head>
<body>
  <!-- ===== ログインページ (login.html) ===== -->
  <div id="loginPage" class="login-page">
    <div class="login-box">
      <h2>交通費申請システム</h2>
      <form id="loginForm">
        <label for="user_id">ID</label>
        <input type="text" id="user_id" placeholder="学籍番号または管理者ID" required>
        <label for="password">パスワード</label>
        <input type="password" id="password" placeholder="パスワードを入力" required>
        <div class="user-type-toggle">
          <label><input type="radio" name="user_type" value="user" checked> 一般ユーザー</label>
          <label><input type="radio" name="user_type" value="admin"> 管理者</label>
        </div>
        <button type="submit">ログイン</button>
      </form>
    </div>
  </div>

  <!-- ===== 一般ユーザーフォーム (form.html) ===== -->
  <div id="formPage" class="container" style="display: none;">
    <h1>交通費申請フォーム</h1>
    <form id="requestForm">
      <p id="userInfo"></p>
      <label for="date">利用日:</label>
      <input type="date" id="date" required>
      <label for="from">出発地:</label>
      <input type="text" id="from" required>
      <label for="to">到着地:</label>
      <input type="text" id="to" required>
      <label>区間:</label>
      <select id="direction" required>
        <option value="">選択してください</option>
        <option value="往路">往路</option>
        <option value="復路">復路</option>
      </select>
      <button type="submit">送信</button>
    </form>
    <div id="result"></div>
  </div>

  <!-- ===== 管理者ページ (admin.html) ===== -->
  <div id="adminPage" style="display: none;">
    <header>
      <h1>管理者ページ</h1>
      <button onclick="logout()">ログアウト</button>
    </header>
    <main>
      <section>
        <h2>申請データプレビュー</h2>
        <button onclick="downloadCSV()">CSV出力</button>
        <table id="dataTable">
          <thead>
            <tr>
              <th>学籍番号</th>
              <th>氏名</th>
              <th>日付</th>
              <th>出発地</th>
              <th>到着地</th>
              <th>区間</th>
            </tr>
          </thead>
          <tbody></tbody>
        </table>
      </section>
    </main>
  </div>

  <script>
    // ===== login.js =====
    const userMembers = [
      { id: "B23317", password: "1234", name: "藤本健太" },
      { id: "23456789", password: "user456", name: "佐藤花子" },
      { id: "34567890", password: "user789", name: "鈴木次郎" }
    ];
    const adminMembers = [
      { id: "B23307", password: "1234", name: "鎌谷篤志" },
      { id: "admin2", password: "admin456", name: "事務局担当者" }
    ];

    document.getElementById("loginForm").addEventListener("submit", function(e) {
      e.preventDefault();
      const userId = document.getElementById("user_id").value.trim();
      const password = document.getElementById("password").value.trim();
      const isAdmin = document.querySelector('input[name="user_type"]:checked').value === "admin";

      if (!userId || !password) {
        alert("IDとパスワードを入力してください。");
        return;
      }

      let authenticatedUser = null;
      let userType = "";

      if (isAdmin) {
        authenticatedUser = adminMembers.find(admin => admin.id === userId && admin.password === password);
        userType = "admin";
      } else {
        authenticatedUser = userMembers.find(user => user.id === userId && user.password === password);
        userType = "user";
      }

      if (authenticatedUser) {
        sessionStorage.setItem("user_type", userType);
        sessionStorage.setItem("user_id", authenticatedUser.id);
        sessionStorage.setItem("user_name", authenticatedUser.name);
        
        if (userType === "admin") {
          document.getElementById("loginPage").style.display = "none";
          document.getElementById("adminPage").style.display = "block";
          renderApplications();
        } else {
          document.getElementById("loginPage").style.display = "none";
          document.getElementById("formPage").style.display = "block";
          document.getElementById("userInfo").innerHTML = `【${authenticatedUser.id}】${authenticatedUser.name} さん`;
        }
      } else {
        alert(`${isAdmin ? "管理者" : "ユーザー"}IDまたはパスワードが間違っています。`);
      }
    });

    // ===== form.js =====
    document.getElementById("requestForm").addEventListener("submit", function(e) {
      e.preventDefault();
      const formData = {
        date: document.getElementById("date").value,
        from: document.getElementById("from").value,
        to: document.getElementById("to").value,
        direction: document.getElementById("direction").value
      };

      if (!formData.date || !formData.from || !formData.to || !formData.direction) {
        alert("すべての項目を入力してください。");
        return;
      }

      const result = `
        <div class="result-box">
          <p><strong>送信完了！</strong></p>
          <p>学籍番号: ${sessionStorage.getItem("user_id")}</p>
          <p>氏名: ${sessionStorage.getItem("user_name")}</p>
          <p>日付: ${formData.date}</p>
          <p>出発地: ${formData.from}</p>
          <p>到着地: ${formData.to}</p>
          <p>区間: ${formData.direction}</p>
        </div>
      `;
      document.getElementById("result").innerHTML = result;
      this.reset();
    });

    // ===== admin.js =====
    let applications = []; // 空の配列で初期化

    function renderApplications() {
      const tbody = document.querySelector("#dataTable tbody");
      tbody.innerHTML = "";
      
      if (applications.length === 0) {
        const row = document.createElement("tr");
        row.innerHTML = `<td colspan="6" style="text-align: center;">申請データがありません</td>`;
        tbody.appendChild(row);
      } else {
        applications.forEach(app => {
          const row = document.createElement("tr");
          row.innerHTML = `
            <td>${app.id}</td>
            <td>${app.name}</td>
            <td>${app.date}</td>
            <td>${app.from}</td>
            <td>${app.to}</td>
            <td>${app.direction}</td>
          `;
          tbody.appendChild(row);
        });
      }
    }

    function downloadCSV() {
      if (applications.length === 0) {
        alert("出力するデータがありません");
        return;
      }

      let csv = "学籍番号,氏名,日付,出発地,到着地,区間\n";
      applications.forEach(app => {
        csv += `${app.id},${app.name},${app.date},${app.from},${app.to},${app.direction}\n`;
      });
      const blob = new Blob(["\uFEFF" + csv], { type: "text/csv;charset=utf-8;" });
      const url = URL.createObjectURL(blob);
      const link = document.createElement("a");
      link.setAttribute("href", url);
      link.setAttribute("download", `交通費申請_${new Date().toISOString().slice(0,10)}.csv`);
      link.click();
    }

    function logout() {
      sessionStorage.clear();
      document.getElementById("loginPage").style.display = "flex";
      document.getElementById("formPage").style.display = "none";
      document.getElementById("adminPage").style.display = "none";
      document.getElementById("user_id").value = "";
      document.getElementById("password").value = "";
    }

    // 初期表示（ログイン済みかチェック）
    window.addEventListener("DOMContentLoaded", () => {
      const userType = sessionStorage.getItem("user_type");
      const userId = sessionStorage.getItem("user_id");
      
      if (userId) {
        if (userType === "admin") {
          document.getElementById("loginPage").style.display = "none";
          document.getElementById("adminPage").style.display = "block";
          renderApplications();
        } else {
          document.getElementById("loginPage").style.display = "none";
          document.getElementById("formPage").style.display = "block";
          document.getElementById("userInfo").innerHTML = `【${userId}】${sessionStorage.getItem("user_name")} さん`;
        }
      }
    });
  </script>
</body>
</html>

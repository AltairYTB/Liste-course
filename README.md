<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Liste de Courses Avancée</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(135deg, #f7971e, #ffd200);
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
      transition: background-color 0.3s;
    }

    .container {
      background-color: #fff;
      padding: 30px 20px;
      border-radius: 20px;
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
      width: 90%;
      max-width: 420px;
      text-align: center;
    }

    h1 {
      margin-bottom: 20px;
      color: #333;
    }

    input[type="text"] {
      width: 100%;
      padding: 12px;
      font-size: 16px;
      border-radius: 10px;
      border: 1px solid #ccc;
      margin-bottom: 15px;
    }

    ul {
      list-style: none;
      padding: 0;
      margin-top: 10px;
    }

    li {
      background: #f3f3f3;
      margin: 5px 0;
      padding: 10px 12px;
      border-radius: 10px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    li.checked span {
      text-decoration: line-through;
      color: #888;
    }

    .actions {
      margin-top: 15px;
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      justify-content: center;
    }

    button {
      background-color: #f7971e;
      border: none;
      color: white;
      padding: 10px 14px;
      border-radius: 8px;
      cursor: pointer;
      font-size: 14px;
      transition: background 0.3s;
    }

    button:hover {
      background-color: #e57f00;
    }

    .delete {
      background-color: #ff4d4d;
    }

    .delete:hover {
      background-color: #d93636;
    }

    .priority {
      margin-left: 10px;
      padding: 5px 8px;
      border-radius: 5px;
    }

    .priority-high {
      background-color: #f44336;
      color: white;
    }

    .priority-medium {
      background-color: #ff9800;
      color: white;
    }

    .priority-low {
      background-color: #4caf50;
      color: white;
    }

    .mode-dark {
      background-color: #333;
      color: white;
    }

    .mode-dark input[type="text"], .mode-dark button {
      background-color: #555;
      color: white;
      border: 1px solid #444;
    }

    .mode-dark li {
      background-color: #444;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>🛒 Liste de Courses Avancée</h1>
    <input type="text" id="itemInput" placeholder="Ajouter un article..." />
    
    <div class="actions">
      <button onclick="toggleAll()">✅ Tout cocher/décocher</button>
      <button class="delete" onclick="clearAll()">🗑 Tout effacer</button>
      <button onclick="sortList()">🔠 Trier A-Z</button>
      <button onclick="filterChecked(true)">✅ Articles cochés</button>
      <button onclick="filterChecked(false)">❌ Articles non cochés</button>
      <button onclick="exportList()">📥 Exporter la liste</button>
      <button onclick="toggleMode()">🌙/🌞 Mode Sombre</button>
    </div>

    <ul id="list"></ul>
  </div>

  <script>
    const input = document.getElementById("itemInput");
    const list = document.getElementById("list");

    let items = JSON.parse(localStorage.getItem("courses")) || [];
    let darkMode = localStorage.getItem("darkMode") === "true";

    function saveItems() {
      localStorage.setItem("courses", JSON.stringify(items));
    }

    function renderItems() {
      list.innerHTML = "";
      items.forEach((item, index) => {
        const li = document.createElement("li");
        if (item.checked) li.classList.add("checked");

        const priorityClass = item.priority ? `priority-${item.priority}` : "";
        li.innerHTML = `
          <span onclick="toggleItem(${index})">${item.text}</span>
          <span class="priority ${priorityClass}">${item.priority || 'Faible'}</span>
          <button class="delete" onclick="deleteItem(${index})">🗑</button>
        `;
        list.appendChild(li);
      });
    }

    function addItem(text) {
      if (text.trim() === "") return;
      const priority = prompt("Définir la priorité (faible, moyenne, élevée) :").toLowerCase();
      items.push({ text: text.trim(), checked: false, priority: priority || "faible" });
      saveItems();
      renderItems();
      input.value = "";
    }

    function toggleItem(index) {
      items[index].checked = !items[index].checked;
      saveItems();
      renderItems();
    }

    function deleteItem(index) {
      items.splice(index, 1);
      saveItems();
      renderItems();
    }

    function clearAll() {
      if (confirm("Supprimer toute la liste ?")) {
        items = [];
        saveItems();
        renderItems();
      }
    }

    function toggleAll() {
      const allChecked = items.every(item => item.checked);
      items = items.map(item => ({ ...item, checked: !allChecked }));
      saveItems();
      renderItems();
    }

    function sortList() {
      items.sort((a, b) => a.text.localeCompare(b.text));
      saveItems();
      renderItems();
    }

    function filterChecked(isChecked) {
      const filteredItems = items.filter(item => item.checked === isChecked);
      renderFilteredItems(filteredItems);
    }

    function renderFilteredItems(filteredItems) {
      list.innerHTML = "";
      filteredItems.forEach(item => {
        const li = document.createElement("li");
        if (item.checked) li.classList.add("checked");

        const priorityClass = item.priority ? `priority-${item.priority}` : "";
        li.innerHTML = `
          <span>${item.text}</span>
          <span class="priority ${priorityClass}">${item.priority || 'Faible'}</span>
        `;
        list.appendChild(li);
      });
    }

    function exportList() {
      const text = items.map(item => `${item.text} - ${item.priority}`).join("\n");
      const blob = new Blob([text], { type: "text/plain;charset=utf-8" });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "list_de_courses.txt";
      a.click();
    }

    function toggleMode() {
      darkMode = !darkMode;
      localStorage.setItem("darkMode", darkMode);
      document.body.classList.toggle("mode-dark", darkMode);
      renderItems();
    }

    input.addEventListener("keydown", (e) => {
      if (e.key === "Enter") addItem(input.value);
    });

    // Activer ou désactiver le mode sombre à l'ouverture
    if (darkMode) {
      document.body.classList.add("mode-dark");
    }

    renderItems();
  </script>
</body>
</html>

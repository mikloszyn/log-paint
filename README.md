<!DOCTYPE html>
<html lang="pl">
<head>
  <meta charset="UTF-8">
  <title>Paint z kontami i pytaniem o zapis</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #eef;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding-top: 30px;
    }
    .box {
      background-color: white;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.2);
      text-align: center;
      margin-bottom: 20px;
    }
    input, select {
      margin: 5px;
      padding: 10px;
      width: 200px;
      border-radius: 5px;
      border: 1px solid #ccc;
    }
    button {
      padding: 10px 15px;
      margin: 5px;
      background-color: #4CAF50;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }
    canvas {
      border: 2px solid #333;
      background-color: white;
      cursor: crosshair;
    }
    #paintArea {
      display: none;
      text-align: center;
    }
  </style>
</head>
<body>
  <div class="box" id="authBox">
    <h2>Logowanie / Rejestracja</h2>
    <input type="text" id="username" placeholder="Nazwa użytkownika"><br>
    <input type="password" id="password" placeholder="Hasło"><br>
    <button onclick="login()">Zaloguj</button>
    <button onclick="register()">Utwórz konto</button>
    <div id="authMessage"></div>
  </div>

  <div id="paintArea">
    <h2>🎨 Paint</h2>
    <div>
      <label for="color">Kolor:</label>
      <select id="color">
        <option value="#000000">Czarny</option>
        <option value="#ff0000">Czerwony</option>
        <option value="#00ff00">Zielony</option>
        <option value="#0000ff">Niebieski</option>
        <option value="#ffff00">Żółty</option>
      </select>

      <label for="customColor">Lub wybierz własny:</label>
      <input type="color" id="customColor" value="#000000">

      <label for="brush">Pędzel:</label>
      <select id="brush">
        <option value="circle">Okrągły</option>
        <option value="square">Kwadratowy</option>
        <option value="line">Linia</option>
      </select>

      <button onclick="toggleEraser()">Gumka</button>
      <button onclick="clearCanvas()">Wyczyść</button>
      <button onclick="saveDrawing()">Zapisz</button>
      <button onclick="logout()">Wyloguj się</button>
    </div>

    <canvas id="canvas" width="600" height="400"></canvas>

    <h3>📁 Twoje rysunki:</h3>
    <select id="savedList" onchange="loadDrawing()"></select>
  </div>

  <script>
    let currentUser = null;
    let painting = false;
    let eraser = false;
    let drawingSaved = true;

    const canvas = document.getElementById("canvas");
    const ctx = canvas.getContext("2d");

    function register() {
      const user = username.value;
      const pass = password.value;
      if (localStorage.getItem("user_" + user)) {
        authMessage.innerText = "Użytkownik już istnieje.";
      } else {
        localStorage.setItem("user_" + user, pass);
        localStorage.setItem("drawings_" + user, JSON.stringify([]));
        authMessage.innerText = "Konto utworzone!";
      }
    }

    function login() {
      const user = username.value;
      const pass = password.value;
      if (localStorage.getItem("user_" + user) === pass) {
        currentUser = user;
        authBox.style.display = "none";
        paintArea.style.display = "block";
        loadSavedList();
      } else {
        authMessage.innerText = "Błędna nazwa lub hasło.";
      }
    }

    function logout() {
      if (!drawingSaved) {
        const confirmSave = confirm("Czy zapisać rysunek przed wyjściem?");
        if (confirmSave) {
          saveDrawing();
        }
      }
      currentUser = null;
      authBox.style.display = "block";
      paintArea.style.display = "none";
      clearCanvas();
      drawingSaved = true;
    }

    canvas.addEventListener("mousedown", () => painting = true);
    canvas.addEventListener("mouseup", () => painting = false);
    canvas.addEventListener("mouseleave", () => painting = false);
    canvas.addEventListener("mousemove", draw);

    function draw(e) {
      if (!painting) return;
      drawingSaved = false;
      const rect = canvas.getBoundingClientRect();
      const x = e.clientX - rect.left;
      const y = e.clientY - rect.top;

      const selectedColor = eraser ? "#ffffff" : (customColor.value || color.value);
      ctx.fillStyle = selectedColor;
      const brush = brushSelect.value;

      if (brush === "circle") {
        ctx.beginPath();
        ctx.arc(x, y, 5, 0, Math.PI * 2);
        ctx.fill();
      } else if (brush === "square") {
        ctx.fillRect(x - 5, y - 5, 10, 10);
      } else if (brush === "line") {
        ctx.beginPath();
        ctx.moveTo(x - 10, y);
        ctx.lineTo(x + 10, y);
        ctx.strokeStyle = ctx.fillStyle;
        ctx.lineWidth = 3;
        ctx.stroke();
      }
    }

    function toggleEraser() {
      eraser = !eraser;
    }

    function clearCanvas() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawingSaved = false;
    }

    function saveDrawing() {
      const dataURL = canvas.toDataURL();
      const drawings = JSON.parse(localStorage.getItem("drawings_" + currentUser));
      const name = prompt("Podaj nazwę rysunku:");
      if (name) {
        drawings.push({ name, data: dataURL });
        localStorage.setItem("drawings_" + currentUser, JSON.stringify(drawings));
        loadSavedList();
        drawingSaved = true;
      }
    }

    function loadSavedList() {
      const list = document.getElementById("savedList");
      list.innerHTML = "";
      const drawings = JSON.parse(localStorage.getItem("drawings_" + currentUser));
      drawings.forEach((d, i) => {
        const option = document.createElement("option");
        option.value = i;
        option.text = d.name;
        list.appendChild(option);
      });
    }

    function loadDrawing() {
      clearCanvas();
      const index = savedList.value;
      const drawings = JSON.parse(localStorage.getItem("drawings_" + currentUser));
      const img = new Image();
      img.src = drawings[index].data;
      img.onload = () => ctx.drawImage(img, 0, 0);
      drawingSaved = true;
    }

    const brushSelect = document.getElementById("brush");
    const customColor = document.getElementById("customColor");
  </script>
</body>
</html>

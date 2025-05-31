<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>Whiteframe</title>
  <style>
    body {
      background-color: #000;
      color: #fff;
      font-family: sans-serif;
      text-align: center;
      padding: 20px;
    }

    input, select, button {
      margin: 10px;
    }

    canvas {
      margin-top: 20px;
      max-width: 100%;
      height: auto;
      border: 1px solid #444;
      background-color: #222;
    }

    #orientationToggle.active {
      background-color: #4caf50;
      color: #fff;
    }

    #downloadLink {
      display: none;
      margin-top: 15px;
      color: #4caf50;
      text-decoration: underline;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <h1>Whiteframe</h1>

  <input type="file" id="imageInput" accept="image/*"><br>

  <label>枠の太さ:</label>
  <select id="frameSize">
    <option value="50">50px</option>
    <option value="100" selected>100px</option>
    <option value="200">200px</option>
    <option value="custom">カスタム</option>
  </select>
  <input type="number" id="customSize" placeholder="数値入力" style="display:none; width:80px;"> px<br>

  <label>背景紙の比率:</label>
  <select id="canvasRatio">
    <option value="">なし</option>
    <option value="1">1:1</option>
    <option value="3/2">3:2</option>
    <option value="4/3">4:3</option>
    <option value="16/9">16:9</option>
  </select>
  <button id="orientationToggle">縦</button><br>

  <button id="addFrameBtn">枠を追加</button><br>
  <canvas id="previewCanvas"></canvas><br>
  <a id="downloadLink" download>画像を保存（長押しで保存可）</a>

  <script>
    const imageInput = document.getElementById("imageInput");
    const frameSize = document.getElementById("frameSize");
    const customSize = document.getElementById("customSize");
    const canvasRatio = document.getElementById("canvasRatio");
    const orientationToggle = document.getElementById("orientationToggle");
    const addFrameBtn = document.getElementById("addFrameBtn");
    const previewCanvas = document.getElementById("previewCanvas");
    const ctx = previewCanvas.getContext("2d");
    const downloadLink = document.getElementById("downloadLink");

    let image = new Image();
    let isPortrait = true;
    let imageFileName = "framed";

    orientationToggle.addEventListener("click", () => {
      isPortrait = !isPortrait;
      orientationToggle.classList.toggle("active", isPortrait);
      orientationToggle.textContent = isPortrait ? "縦" : "横";
    });

    frameSize.addEventListener("change", () => {
      customSize.style.display = frameSize.value === "custom" ? "inline-block" : "none";
    });

    imageInput.addEventListener("change", (e) => {
      const file = e.target.files[0];
      if (file) {
        imageFileName = file.name.replace(/\.[^/.]+$/, "") + "_01";
        const reader = new FileReader();
        reader.onload = (event) => {
          image.onload = () => drawImage();
          image.src = event.target.result;
        };
        reader.readAsDataURL(file);
      }
    });

    addFrameBtn.addEventListener("click", () => {
      if (!image.src) return;

      const padding = frameSize.value === "custom" ? parseInt(customSize.value, 10) || 0 : parseInt(frameSize.value, 10);
      const ratio = canvasRatio.value;

      let bgWidth = image.width + padding * 2;
      let bgHeight = image.height + padding * 2;

      if (ratio) {
        const parsedRatio = eval(ratio);
        if (isPortrait) {
          bgHeight = Math.max(bgHeight, Math.round(bgWidth / parsedRatio));
        } else {
          bgWidth = Math.max(bgWidth, Math.round(bgHeight * parsedRatio));
        }
      }

      previewCanvas.width = bgWidth;
      previewCanvas.height = bgHeight;

      ctx.fillStyle = "#fff";
      ctx.fillRect(0, 0, bgWidth, bgHeight);

      const x = (bgWidth - image.width) / 2;
      const y = (bgHeight - image.height) / 2;
      ctx.drawImage(image, x, y);

      downloadLink.href = previewCanvas.toDataURL("image/jpeg");
      downloadLink.download = imageFileName + ".jpg";
      downloadLink.style.display = "inline-block";
    });
  </script>
</body>
</html>

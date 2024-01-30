<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>TrackLive</title>
  <script src="https://unpkg.com/wavesurfer.js"></script>
  <style>
    body {
      font-family: 'Arial', sans-serif;
    }

    h2 {
      font-weight: bold;
      font-size: 24px;
      color: black;
    }

    #subtitle {
      font-size: 16px;
      font-weight: normal;
      color: gray;
    }

    button {
      font-size: 16px;
      font-weight: bold;
      padding: 10px;
      cursor: pointer;
      border: none;
    }

    #playButton {
      color: black;
    }

    #pauseButton {
      color: gray;
    }

    #dropArea {
      border: 2px dashed #ccc;
      padding: 20px;
      text-align: center;
    }

    #uploadButton {
      font-size: 16px;
      font-weight: bold;
      padding: 10px;
      cursor: pointer;
      border: none;
      background-color: #4CAF50;
      color: white;
    }

    #fileInput {
      display: none;
    }
  </style>
</head>
<body>

<h2>TrackLive</h2>
<p id="subtitle">By TZU Worship</p>

<div id="dropArea" onclick="selectFiles()">
  Haz clic aquí o arrastra y suelta tus archivos de audio.
  <br>
  <button type="button" id="uploadButton" onclick="submitForm()">Subir</button>
  <input type="file" id="fileInput" multiple onchange="handleFileSelect(this)">
</div>

<button id="playButton" onclick="playAll()">Play</button>
<button id="pauseButton" onclick="pauseAll()">Pause</button>

<div>
  <label for="masterVolume">Volumen Maestro:</label>
  <input type="range" id="masterVolume" min="0" max="1" step="0.1" value="1" oninput="changeMasterVolume(this.value)">
</div>

<div>
  <label for="masterSeek">Posición Maestra:</label>
  <input type="range" id="masterSeek" min="0" max="100" step="0.1" value="0" oninput="changeMasterPosition(this.value)">
  <div id="masterWaveform"></div>
</div>

<script>
  var audioElements = [];
  var masterVolume = 1;
  var masterWaveform;

  function selectFiles() {
    document.getElementById('fileInput').click();
  }

  function allowDrop(event) {
    event.preventDefault();
    document.getElementById('dropArea').style.border = '2px dashed #aaa';
  }

  function handleFileSelect(input) {
    document.getElementById('dropArea').style.border = '2px dashed #ccc';

    var files = input.files;

    for (var i = 0; i < files.length; i++) {
      var audio = document.createElement('audio');
      audio.src = URL.createObjectURL(files[i]);
      audio.preload = 'auto';
      audio.controls = true;
      audioElements.push(audio);

      // Mostrar el nombre del archivo
      var fileName = document.createElement('p');
      fileName.textContent = files[i].name;
      document.body.appendChild(fileName);

      document.body.appendChild(audio);
    }

    // Inicializar Wavesurfer para la forma de onda maestra
    masterWaveform = WaveSurfer.create({
      container: '#masterWaveform',
      waveColor: 'gray',
      progressColor: 'black',
      height: 50,
      cursorWidth: 0,
      interact: true,
    });

    // Cargar la forma de onda maestra con el primer archivo de audio
    masterWaveform.load(URL.createObjectURL(files[0]));

    // Actualizar la forma de onda maestra a medida que se reproduce
    audioElements.forEach(function(audio) {
      audio.addEventListener('timeupdate', function() {
        var currentTime = audio.currentTime;
        var duration = audio.duration;
        var percentage = (currentTime / duration) * 100;
        masterSeek.value = percentage;
        masterWaveform.seekTo(currentTime / duration);
      });
    });

    // Hacer clic en la forma de onda maestra para cambiar la posición del track maestro
    masterWaveform.on('seek', function(progress) {
      var currentTime = progress * audioElements[0].duration;
      audioElements.forEach(function(audio) {
        audio.currentTime = currentTime;
      });
    });
  }

  function playAll() {
    audioElements.forEach(function(audio) {
      audio.play();
      audio.volume = masterVolume;
    });
  }

  function pauseAll() {
    audioElements.forEach(function(audio) {
      audio.pause();
    });
  }

  function changeMasterVolume(value) {
    masterVolume = parseFloat(value);
    audioElements.forEach(function(audio) {
      audio.volume = masterVolume;
    });
  }

  function changeMasterPosition(value) {
    var percentage = parseFloat(value);
    var currentTime = audioElements[0].duration * (percentage / 100);
    audioElements.forEach(function(audio) {
      audio.currentTime = currentTime;
    });
  }

  function submitForm() {
    // Lógica para manejar la carga de archivos
    console.log("Archivos listos para ser procesados:", audioElements);
  }

  var dropArea = document.getElementById('dropArea');
  dropArea.addEventListener('dragover', allowDrop);
  dropArea.addEventListener('drop', handleFileSelect);
</script>

</body>
</html>

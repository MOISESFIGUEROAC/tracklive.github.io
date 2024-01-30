<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>TrackLive</title>
  <script src="https://unpkg.com/wavesurfer.js"></script>
  <style>
    body {
      font-family: 'Arial', sans-serif; /* Cambiar a la fuente que desees */
    }

    h2 {
      font-weight: bold;
      font-size: 24px;
      color: black; /* Color del título */
    }

    #subtitle {
      font-size: 16px;
      font-weight: normal;
      color: gray; /* Color del subtítulo */
    }

    button {
      font-size: 16px;
      font-weight: bold;
      padding: 10px;
      cursor: pointer;
      border: none;
    }

    #playButton {
      color: black; /* Color del botón de reproducción */
    }

    #pauseButton {
      color: gray; /* Color del botón de pausa */
    }

    #masterSeek {
      width: 100%;
      margin-top: 10px;
      -webkit-appearance: none; /* Remove default styles in WebKit browsers */
      appearance: none;
      height: 10px;
      background: #ddd; /* Fondo del riel del control deslizante */
      outline: none;
      opacity: 0.7;
      -webkit-transition: 0.2s; /* Transición suave para cambios */
      transition: opacity 0.2s;
      border-radius: 5px; /* Bordes redondeados */
    }

    #masterSeek:hover {
      opacity: 1; /* Hacer el control deslizante más visible al pasar el mouse */
    }

    #masterSeek::-webkit-slider-thumb {
      -webkit-appearance: none; /* Remove default styles in WebKit browsers */
      appearance: none;
      width: 20px; /* Ancho del pulgar del control deslizante */
      height: 20px; /* Altura del pulgar del control deslizante */
      background: #3498db; /* Color del pulgar del control deslizante */
      cursor: pointer;
      border-radius: 50%; /* Bordes redondeados para el pulgar del control deslizante */
    }

    #masterSeek::-moz-range-thumb {
      width: 20px; /* Ancho del pulgar del control deslizante en navegadores Firefox */
      height: 20px; /* Altura del pulgar del control deslizante en navegadores Firefox */
      background: #3498db; /* Color del pulgar del control deslizante en navegadores Firefox */
      cursor: pointer;
      border-radius: 50%; /* Bordes redondeados para el pulgar del control deslizante en navegadores Firefox */
    }
  </style>
</head>
<body>

<h2>TrackLive</h2>
<p id="subtitle">By TZU Worship</p>

<div id="dropArea" style="border: 2px dashed #ccc; padding: 20px; text-align: center;">
  Arrastra y suelta tus archivos de audio aquí.
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

  function allowDrop(event) {
    event.preventDefault();
    document.getElementById('dropArea').style.border = '2px dashed #aaa';
  }

  function drop(event) {
    event.preventDefault();
    document.getElementById('dropArea').style.border = '2px dashed #ccc';

    var files = event.dataTransfer.files;

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
      waveColor: 'gray', // Cambiar a gris
      progressColor: 'black', // Cambiar a negro
      height: 50,
      cursorWidth: 0,
      interact: true,  // Permitir interacción con la forma de onda
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

  var dropArea = document.getElementById('dropArea');
  dropArea.addEventListener('dragover', allowDrop);
  dropArea.addEventListener('drop', drop);
</script>

</body>
</html>

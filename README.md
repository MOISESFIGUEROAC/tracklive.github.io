<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>TrackZonaUngida</title>

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

    .soloButton {
      background-color: white;
      color: black;
      border: 1px solid black;
      padding: 5px;
      cursor: pointer;
    }

    .soloButton.active {
      background-color: black;
      color: white;
    }

    #playButton.active,
    #pauseButton.active,
    #stopButton.active,
    #openButton.active {
      background-color: black;
      color: white;
    }

    #dropArea {
      border: 2px dashed #ccc;
      padding: 20px;
      text-align: center;
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    p {
      margin: 5px 0;
    }

    #masterSeek {
      width: 100%;
      margin-top: 10px;
      -webkit-appearance: none;
      appearance: none;
      height: 10px;
      background: #ddd;
      outline: none;
      opacity: 0.7;
      transition: opacity 0.2s;
      border-radius: 5px;
    }

    #masterSeek:hover {
      opacity: 1;
    }

    #masterSeek::-webkit-slider-thumb,
    #masterSeek::-moz-range-thumb {
      -webkit-appearance: none;
      appearance: none;
      width: 20px;
      height: 20px;
      background: #3498db;
      cursor: pointer;
      border-radius: 50%;
    }
  </style>
</head>
<body>

<h2>TrackZonaUngida</h2>
<p id="subtitle">By TZU Worship</p>

<div id="dropArea">
  Arrastra y suelta tus archivos de audio aquí.
</div>

<button id="playButton" onclick="playAll()">Play</button>
<button id="pauseButton" onclick="pauseAll()">Pause</button>
<button id="stopButton" onclick="stopAll()">Stop</button> <!-- Nuevo botón "Stop" -->
<button id="openButton" onclick="openAll()">Open</button> <!-- Nuevo botón "Open" -->

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
  var soloButtons = [];
  var openButtonClicked = false; // Variable para rastrear si se hizo clic en el botón "Open"

  function allowDrop(event) {
    event.preventDefault();
    updateDropAreaStyle('2px dashed #aaa');
  }

  function drop(event) {
    event.preventDefault();
    updateDropAreaStyle('2px dashed #ccc');

    var files = event.dataTransfer.files;

    for (var i = 0; i < files.length; i++) {
      createAudioElement(files[i]);
    }

    initializeMasterWaveform(files);
    updateMasterWaveform(files[0]);

    updateAudioElementsTimeUpdate();
    setMasterWaveformSeek();
  }

  function updateDropAreaStyle(style) {
    document.getElementById('dropArea').style.border = style;
  }

  function createAudioElement(file) {
    var audio = document.createElement('audio');
    audio.src = URL.createObjectURL(file);
    audio.preload = 'auto';
    audio.controls = true;
    audioElements.push(audio);

    var fileName = document.createElement('p');
    fileName.textContent = file.name;

    var soloButton = document.createElement('button');
    soloButton.textContent = 'Solo';
    soloButton.className = 'soloButton';
    soloButton.onclick = function() {
      toggleSolo(audio);
    };
    soloButtons.push({ audio: audio, button: soloButton });

    document.body.appendChild(fileName);
    document.body.appendChild(audio);
    document.body.appendChild(soloButton);
  }

  function initializeMasterWaveform(files) {
    masterWaveform = WaveSurfer.create({
      container: '#masterWaveform',
      waveColor: 'gray',
      progressColor: 'black',
      height: 50,
      cursorWidth: 0,
      interact: true,
    });
  }

  function updateMasterWaveform(file) {
    masterWaveform.load(URL.createObjectURL(file));
  }

  function updateAudioElementsTimeUpdate() {
    audioElements.forEach(function(audio) {
      audio.addEventListener('timeupdate', function() {
        var currentTime = audio.currentTime;
        var duration = audio.duration;
        var percentage = (currentTime / duration) * 100;
        document.getElementById('masterSeek').value = percentage;
        masterWaveform.seekTo(currentTime / duration);
      });
    });
  }

  function setMasterWaveformSeek() {
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
      document.getElementById('playButton').classList.add('active');
      document.getElementById('pauseButton').classList.remove('active');
      document.getElementById('stopButton').classList.remove('active');
    });
  }

  function pauseAll() {
    audioElements.forEach(function(audio) {
      audio.pause();
      document.getElementById('pauseButton').classList.add('active');
      document.getElementById('playButton').classList.remove('active');
      document.getElementById('stopButton').classList.remove('active');
    });
  }

  function stopAll() {
    audioElements.forEach(function(audio) {
      audio.pause();
      audio.currentTime = 0;
      document.getElementById('stopButton').classList.add('active');
      document.getElementById('playButton').classList.remove('active');
      document.getElementById('pauseButton').classList.remove('active');
    });
  }

  function openAll() {
    soloButtons.forEach(function(soloButton) {
      soloButton.button.classList.remove('active');
    });
    audioElements.forEach(function(audio) {
      audio.volume = masterVolume;
    });

    // Alternar entre blanco y negro para el botón "Open"
    openButtonClicked = !openButtonClicked;
    var openButton = document.getElementById('openButton');
    openButton.classList.toggle('active', openButtonClicked);

    document.getElementById('playButton').classList.remove('active');
    document.getElementById('pauseButton').classList.remove('active');
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

  function toggleSolo(clickedAudio) {
    var soloButton = soloButtons.find(function(solo) {
      return solo.audio === clickedAudio;
    });

    if (soloButton) {
      soloButton.button.classList.toggle('active');

      var activeSolos = soloButtons.filter(function(solo) {
        return solo.button.classList.contains('active');
      });

      if (activeSolos.length > 0) {
        audioElements.forEach(function(audio) {
          audio.volume = activeSolos.some(function(solo) {
            return solo.audio === audio;
          }) ? masterVolume : 0;
        });
      } else {
        audioElements.forEach(function(audio) {
          audio.volume = masterVolume;
        });
      }
    }
  }

  var dropArea = document.getElementById('dropArea');
  dropArea.addEventListener('dragover', allowDrop);
  dropArea.addEventListener('drop', drop);
</script>

</body>
</html>

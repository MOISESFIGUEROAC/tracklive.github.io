<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Combine Audio</title>
</head>
<body>
    <h1>Combine Audio</h1>
    <input type="file" id="audio_files" accept=".mp3, .wav" multiple>
    <button onclick="combineAudio()">Exportar</button>

    <div id="preview-container" style="display:none">
        <h2>Vista Previa</h2>
        <audio controls id="audio-preview"></audio>
    </div>

    <div id="progress-container" style="display:none">
        <label for="progress">Progreso:</label>
        <progress id="progress" value="0" max="100"></progress>
    </div>

    <a id="downloadLink" style="display:none">Descargar Audio Combinado</a>

    <script>
        async function combineAudio() {
            var input = document.getElementById('audio_files');
            var files = input.files;

            if (files.length > 0) {
                var audioContext = new (window.AudioContext || window.webkitAudioContext)();
                var audioBufferList = [];

                async function readFile(file) {
                    return new Promise((resolve, reject) => {
                        var reader = new FileReader();

                        reader.onload = function (e) {
                            audioContext.decodeAudioData(e.target.result, function (buffer) {
                                audioBufferList.push(buffer);
                                resolve();
                            }, reject);
                        };

                        reader.readAsArrayBuffer(file);
                    });
                }

                function mergeBuffers(audioBufferList) {
                    var totalLength = audioBufferList.reduce((acc, buffer) => acc + buffer.length, 0);
                    var mergedBuffer = audioContext.createBuffer(1, totalLength, audioBufferList[0].sampleRate);
                    var offset = 0;

                    audioBufferList.forEach(function (buffer) {
                        mergedBuffer.getChannelData(0).set(buffer.getChannelData(0), offset);
                        offset += buffer.length;
                    });

                    return mergedBuffer;
                }

                // Show progress bar
                var progressContainer = document.getElementById('progress-container');
                progressContainer.style.display = 'block';

                var progressBar = document.getElementById('progress');

                try {
                    await Promise.all(files.map(file => readFile(file)));

                    var combinedBuffer = mergeBuffers(audioBufferList);

                    // Create a new Blob with the combined audio data
                    var audioBlob = new Blob([combinedBuffer.getChannelData(0).buffer], { type: 'audio/wav' });

                    // Create a download link and trigger the download
                    var downloadLink = document.getElementById('downloadLink');
                    downloadLink.href = URL.createObjectURL(audioBlob);
                    downloadLink.download = 'combined_audio.wav';
                    downloadLink.style.display = 'block';

                    // Hide progress bar
                    progressContainer.style.display = 'none';

                    // Show preview container
                    var previewContainer = document.getElementById('preview-container');
                    previewContainer.style.display = 'block';

                    // Set the source of the audio preview
                    var audioPreview = document.getElementById('audio-preview');
                    audioPreview.src = URL.createObjectURL(audioBlob);
                    audioPreview.load();
                } catch (error) {
                    console.error(error);
                } finally {
                    // Close the audio context
                    if (audioContext.state !== 'closed') {
                        audioContext.close();
                    }
                }
            }
        }
    </script>
</body>
</html>

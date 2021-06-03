<script>
  import { onMount } from "svelte";
  import * as tf from "@tensorflow/tfjs";
  import * as tmImage from "@teachablemachine/image";
 // import Webcam from 'webcam-easy';

  let videoEl;
  let errorMessage;
  let model;
  let loading = true;
  let percentage = "";
  let name = "";

const button = document.getElementById('button');
const select = document.getElementById('select');


  const URL = "model00/";
  const modelURL = URL + "model.json";
  const metadataURL = URL + "metadata.json";

const constraints = { 
    //  video: true, 
    video: {
    width: {
      min: 1280,
      ideal: 1920,
      max: 2560,
    },
    height: {
      min: 720,
      ideal: 1080,
      max: 1440
    },
     facingMode:  { exact: "environment"}
  }
}
  onMount(async () => {
    try {
      model = await tmImage.load(modelURL, metadataURL);
      const stream = await navigator.mediaDevices.getUserMedia(constraints);

      videoEl.srcObject = stream;
      videoEl.play();
      setInterval(predict, 1000);
      loading = false;
    } catch (e) {
      console.error(e, "camera access denied");
      errorMessage = "Kamera nie podłączona i nie działa :(";
    }
  });

  async function predict() {
    const predictions = await model.predict(videoEl);
    const [choosenPrediction] = predictions.sort(
      (a, b) => b.probability - a.probability
    );

    //console.log(predictions);

    if (choosenPrediction) {
      percentage = (choosenPrediction.probability * 100).toFixed(2) + "%";
      name = classNameToLabel(choosenPrediction.className);
    }
  }
  function classNameToLabel(className) {
    switch (className) {
      case "Pik":
        return "kartę Pik ♠";
      case "Kier":
        return "kartę Kier ♥";
      case "Karo":
        return "kartę Karo ♦";
      case "Trefl":
        return "kartę Trefl ♣";
      default:
        return "Nic....";
    }
  }
</script>

<style>
  main {
    width: 95%;
    height: 90vh;
    padding: 0;
    box-sizing: border-box;
    position: absolute;
  }
  video {
    display: block;
    margin: 20px auto;
  }
  h1,
  h2 {
    text-align: center;
  }
  h1 {
    font-size: 40px;
  }
  h2 {
    font-size: 20px;
  }
  .pep {
    text-align: right;
    padding-top: 1rem;
    font-size: 12px;
  }
  a {
    text-decoration: none;
  }
  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>

<main>
  <h1>
    Wyciągnij Asa z rękawa
    <br />
    ♠
    <snap style="color:red;">♦</snap>
    ♣
    <snap style="color:red;">♥</snap>
    ツ 😎
  </h1>
  <video bind:this={videoEl} width="330" height="300" />
  {#if errorMessage}
    <h2 style="color: red;">{errorMessage}</h2>
  {:else if loading}
    <h2>ładuje ... . .</h2>
  {:else if percentage && name}
    <h2>Wykryto {name} w {percentage}</h2>
  {/if}
  <br />
  <br />
    <div class="pep">
    Prosta aplikacja ML/AI, która wykrywa przy pomocy kamery internetowej rodzaj
    <a
      href="https://pl.wikipedia.org/wiki/Karty"
      alt="Wikipedia opis kart do gry"
      rel="noopener noreferrer"
      target="_blank">
      karty do gry.
    </a>
    <br />
    Aplikacja napisana przy użyciu frameworka
    <a
      href="https://svelte.dev"
      alt="Svelte website"
      rel="noopener noreferrer"
      target="_blank">
      Svelte.js
    </a>
    oraz biblioteki
    <a
      href="https://www.tensorflow.org/"
      alt="Website Tensorflow Google"
      rel="noopener noreferrer"
      target="_blank">
      TF.js/tensorflow.keras.
    </a>
    <br />
    Model został wytrenowany na 30 zdjęciach dla każdej z czterech figur/karty,
    w tym wypadku były to Asy.
    <br />
    Model dość dobrze radzi sobie, również z innymi kartami tego samego koloru -
    zapraszam od zabawy ツ
    <br />
    <a
      href="https://trochymiak.net"
      alt="website Piotr Trochymiak"
      rel="noopener noreferrer"
      target="_blank">
      ^p^
    </a>
  </div>
</main>

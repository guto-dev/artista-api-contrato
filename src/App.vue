<script setup>
import { ref } from 'vue';

// Constant to store spotify's API access token
let accessToken = "";

// Consts to manage form
const nome = ref(''); // Event name
const nomeArtista = ref(''); // Artist name (search)
const nomeArtistaSelecionado = ref(''); // Selected artist for select/options
const cacheArtistaSelecionado = ref(''); // Selected artist's cache for storing in input field
const artistas = ref([]); // List of found artists
const dataEvento = ref(''); // Date of the event
const endereco = ref(''); // Address of where the event will occur
const mostrarFormulario = ref(false); // Controls form visibility
const mostrarAlerta = ref(false); // Controls alert visibility

// Const to store Spotify's API credentials
const CLIENT_ID = '85a19b86d7d64765b3ba31799c9ba5a2';
const CLIENT_SECRET = '2ce96af541b64c48a53f016e256ab219';

// Parameters for authentication in spotify's API
var authParameters = {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded'
  },
  body: 'grant_type=client_credentials&client_id=' + CLIENT_ID + '&client_secret=' + CLIENT_SECRET
}

// Generates access token via spotify's API
fetch('https://accounts.spotify.com/api/token', authParameters)
  .then(response => response.json())
  .then(data => {
    accessToken = data.access_token;
    console.log(accessToken);
  })

// Function to search artists through spotify
async function buscarArtistas() {
  // Makes a request to Spotify's API to search for artists through 'nomeArtista'
  const response = await fetch(
    `https://api.spotify.com/v1/search?q=artist:${nomeArtista.value}&type=artist`,
    {
      method: "GET",
      headers: {
        Authorization: `Bearer ${accessToken}`,
        "Content-Type": "application/json",
      },
    }
  );

  // Converts response to json format
  const data = await response.json();

  // Clears artist list and previous display
  artistas.value = [];
  document.querySelector("#container-mostrar-artistas").innerHTML = '';

  // Iterates over found artists and adds them to the list and display
  data.artists.items.forEach((artist) => {
    
    const cacheContratacao = Math.floor(Math.random() * (5000 - 2000 + 1)) + 2000;

    const artistData = {
      id: artist.id,
      name: artist.name,
      cache: cacheContratacao,
    };

    artistas.value.push(artistData);

    const artistCard = document.createElement("div");
    artistCard.className = "card mb-3 mx-auto";
    artistCard.style.maxWidth = "18rem";
    artistCard.innerHTML = `
      <img src="${artist.images[0]?.url || 'https://via.placeholder.com/200x200'}" alt="" class="card-img-top" style="width: 250px; height: 250px;">
      <div class="card-body mx-auto">
        <h5 class="card-title" style="max-width: 200px; word-wrap: break-word; id=${artist.id}">${artist.name}</h5>
        <div class="mb-3" id="finalCard">
          <span>Cache: R$ ${cacheContratacao},00</span>
        </div>
      </div>
    `;
    document.querySelector("#container-mostrar-artistas").appendChild(artistCard);
    nomeArtistaSelecionado.value = artistData.name;
  });

  // Defines form visibility as true so that when the function is called, the form shows up (set as false by default so it doesnt appear at first)
  mostrarFormulario.value = true;
  // Updates cache accordingly to the selected artist in the options
  atualizarCacheArtistaSelecionado();
}

// Function to update artist cache when artist is selected
function atualizarCacheArtistaSelecionado() {
  const artistaSelecionado = artistas.value.find((artista) => artista.name === nomeArtistaSelecionado.value);

  if (artistaSelecionado) {
    cacheArtistaSelecionado.value = `R$ ${artistaSelecionado.cache},00`;
  } else {
    cacheArtistaSelecionado.value = '';
  }
}

// Function to submit form and save data on Local Storage
function submitForm() {
  const dados = {
    nome: nome.value,
    nomeArtistaSelecionado: nomeArtistaSelecionado.value,
    dataEvento: dataEvento.value,
    endereco: endereco.value,
    cacheArtistaSelecionado: cacheArtistaSelecionado.value,
  };

  // Verifies if there is any data on local storage
  let dadosLocalStorage = JSON.parse(localStorage.getItem('dados') || '[]');

  // Adds new data to array
  dadosLocalStorage.push(dados);

  // Saves data on local storage
  localStorage.setItem('dados', JSON.stringify(dadosLocalStorage));

  // Clears all form fields after saving data
  nome.value = '';
  nomeArtistaSelecionado.value = '';
  dataEvento.value = '';
  endereco.value = '';
  cacheArtistaSelecionado.value = '';

  console.log('Dados salvos no Local Storage com sucesso!');
  mostrarAlerta.value = true;
}

// Recovers data from Local Storage
const dadosLocalStorage = JSON.parse(localStorage.getItem('dados') || '[]');
</script>

<template>
  <div>
    <header class="header bg-dark d-flex justify-content-center align-items-center">
      <nav class="navbar">
        <h1 class="title fs-1 text-light">Pesquise pelo seu Artista Favorito</h1>
      </nav>
    </header>
    <div class="container">
      <div class="row">
        <div class="col-sm-12 d-flex justify-content-center">
          <div class="input-group mb-3">
            <input type="text" class="form-control mt-3" v-model="nomeArtista" placeholder="Insira o nome do Artista" aria-label="Insira o nome do Artista" aria-describedby="button-addon2"/>
            <button class="btn btn-outline-secondary mt-3 bg-dark" type="button" id="button-addon2" @click="buscarArtistas">
              Buscar por Artista
            </button>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col-sm-12">
          <div id="container-mostrar-artistas" class="d-flex flex-wrap justify-content-center text-center"></div>
        </div>
      </div>
    </div>

    <!-- Seção para exibir os dados salvos no Local Storage -->
    <div class="row mt-4 pb-3">
      <div class="col-lg-8 col-md-10 col-sm-12 offset-lg-2 offset-md-1">
        <h2 class="mb-4 text-center">Eventos Marcados</h2>
        <div class="list-group">
          <a
            v-for="(dados, index) in dadosLocalStorage"
            :key="index"
            class="list-group-item list-group-item-action"
          >
            <h5 class="mb-3">Nome do Evento: {{ dados.nome }}</h5>
            <p class="mb-1">Artista: {{ dados.nomeArtistaSelecionado }}</p>
            <p class="mb-1">Data do Evento: {{ dados.dataEvento }}</p>
            <p class="mb-1">Endereço: {{ dados.endereco }}</p>
            <p class="mb-1">Cache Artista: {{ dados.cacheArtistaSelecionado }}</p>
          </a>
        </div>
      </div>
    </div>

    <div class="row py-4">
      <div class="col-lg-6 col-md-8 col-sm-12 offset-lg-3 offset-md-2">
        <form @submit.prevent="submitForm" v-if="mostrarFormulario" class="mx-auto max-width-form">
          <div v-if="mostrarAlerta" class="alert alert-success py-3" role="alert">
            Evento registrado com sucesso
          </div>

          <div class="mb-3">
            <label for="nome" class="form-label">Nome</label>
            <input type="text" class="form-control" v-model="nome" id="nome" required>
          </div>
          <div class="mb-3">
            <label for="nomeArtistaSelecionado" class="form-label">Nome do Artista</label>
            <select class="form-select" v-model="nomeArtistaSelecionado" id="nomeArtistaSelecionado" @change="atualizarCacheArtistaSelecionado" required>
              <option value="">Selecione um artista</option>
              <option v-for="artist in artistas" :key="artist.id" :value="artist.name">
                {{ artist.name }}
              </option>
            </select>
          </div>
          <div class="mb-3">
            <label for="dataEvento" class="form-label">Data do Evento</label>
            <input type="date" class="form-control" v-model="dataEvento" id="dataEvento" required>
          </div>
          <div class="mb-3">
            <label for="endereco" class="form-label">Endereço</label>
            <input type="text" class="form-control" v-model="endereco" id="endereco" name="endereco" required>
          </div>
          <div class="mb-3">
            <label for="cache" class="form-label">Cache</label>
            <input type="text" class="form-control" v-model="cacheArtistaSelecionado" id="cache" readonly>
          </div>
          
          <button class="btn bg-dark pb-3" type="submit">Contratar</button>
        </form>
      </div>
    </div>
  </div>

  
</template>

<style scoped>
.header {
  text-align: center;
}

.title {
  font-size: 24px;
}

.search-bar {
  text-align: center;
  margin: 20px 0;
}

.max-width-form {
  max-width: 400px;
  margin: 0 auto;
}


button {
  padding: 11.5px 30px;
  background-color: #007bff;
  color: #fff;
  border: none;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}
</style>

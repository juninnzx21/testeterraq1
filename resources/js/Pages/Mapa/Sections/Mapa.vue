<template>
  <!-- Container do mapa -->
  <div id="map" class="z-[5] h-[calc(100vh)] max-h-[calc(100vh)]"></div>

  <!-- Botão para alternar entre o mapa de rua e o modo satélite -->
  <button @click="toggleMapType" class="custom-button">
    {{ satellite ? 'Mapa de Rua' : 'Satélite' }}
  </button>
</template>

<script setup>
import { ref, onMounted, defineProps } from 'vue';
import 'leaflet';
import 'leaflet/dist/leaflet.css';
import 'proj4';
import 'proj4leaflet';
import 'leaflet-mouse-position';
import 'leaflet.vectorgrid';
import {
  ConfigsLeaflet,
  AdicionaCoordenadasMouse,
  AdicionaEscala,
  FuncaoVisaoInicial,
  AdicionaAttribution,
  CriaMenuContexto,
  FuncaoMapaInformacoes,
  FuncoesZoom
} from '@/Pages/Mapa/Functions/InicializarMapa.js';
import { AdicionaOverlaysPadrao } from '@/Pages/Mapa/Functions/TileLayers/AdicionaOverlaysPadrao.js';
import { ToggleRasterTile } from '@/Pages/Mapa/Functions/TileLayers/ToggleRasterTile.js';

// Define as propriedades do componente
const props = defineProps({
  projeto: Object,
  rasters: Object
});

// Extrai a configuração do mapa do projeto e os rasters
let configsMapa = props.projeto.mapa.configuracao;
let rasters = props.projeto.rasters;
let map = ref(null);
const satellite = ref(false);

// Função para alternar entre o mapa de rua e o modo satélite
function toggleMapType() {
  satellite.value = !satellite.value;
  const rasterName = rasters[0].nome;
  ToggleRaster(rasterName);
}

// Hook de ciclo de vida do Vue - chamado quando o componente é montado
onMounted(() => {
  // Inicializa o mapa Leaflet
  let configs = ConfigsLeaflet(configsMapa);
  map = L.map('map', configs);

  // Define a visão inicial e o zoom
  FuncaoVisaoInicial(map, configsMapa.visaoInicial.x, configsMapa.visaoInicial.y, configsMapa.visaoInicial.z);
  SetaVisaoInicial();
  FuncoesZoom(map);
  FuncaoMapaInformacoes(map);

  // Adiciona coordenadas do mouse se habilitado
  if (configsMapa.funcionalidades.coordenadasMouse) {
    AdicionaCoordenadasMouse(map, configsMapa.configuracoesLeaflet);
  }

  // Desativa o zoom duplo ao clicar
  map.doubleClickZoom.disable();

  // Adiciona a escala se habilitada
  if (configsMapa.funcionalidades.escala) {
    AdicionaEscala(map);
  }

  // Adiciona atribuição se habilitada, remove a atribuição padrão se desabilitada
  if (configsMapa.funcionalidades.atribuicoes) {
    AdicionaAttribution(map, configsMapa.configuracoesLeaflet.atribuicaoPrefixo);
  } else {
    map.attributionControl.remove();
  }

  // Adiciona menu de contexto se habilitado
  if (configsMapa.funcionalidades.menuContexto) {
    CriaMenuContexto(map, configsMapa.configuracoesLeaflet);
  }

  // Adiciona overlays padrão
  AdicionaOverlaysPadrao(map, rasters);

  // Função para alternar uma camada raster específica
  window.ToggleRaster = function (nomeRaster) {
    rasters.forEach(raster => raster.nome == nomeRaster ? ToggleRasterTile(map, raster) : null);
  };

  // Cria e adiciona botões de zoom
  const zoomInBtn = createZoomButton('in');
  const zoomOutBtn = createZoomButton('out');
  map.getContainer().appendChild(zoomInBtn);
  map.getContainer().appendChild(zoomOutBtn);

  // Mostra coordenadas no clique com o botão direito
  map.on('contextmenu', (e) => {
    const popupContent = `<div>
                            <p>Latitude: ${e.latlng.lat}</p>
                            <p>Longitude: ${e.latlng.lng}</p>
                            <p>Data/Hora: ${formatDate(new Date())}</p>
                            </div>`;
    L.popup().setLatLng(e.latlng).setContent(popupContent).openOn(map);
  });
});

// Função para criar botões de zoom
const createZoomButton = (type) => {
  const zoomBtn = L.DomUtil.create('div', 'floating-btn zoom-btn');
  const icon = type === 'in' ? '<circle cx="11" cy="11" r="8"/><line x1="21" x2="16.65" y1="21" y2="16.65"/><line x1="11" x2="11" y1="8" y2="14"/><line x1="8" x2="14" y1="11" y2="11"/>' : '<circle cx="11" cy="11" r="8"/><line x1="21" x2="16.65" y1="21" y2="16.65"/><line x1="8" x2="14" y1="11" y2="11"/>';
  zoomBtn.innerHTML = `<svg xmlns="http://www.w3.org/2000/svg" width="30" height="30" style="p-5 m-5 " viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="">${icon}</svg>`;
  zoomBtn.style.position = 'fixed';
  zoomBtn.style.top = type === 'in' ? '20px' : '70px';
  zoomBtn.style.left = '20px';
  zoomBtn.style.zIndex = '1001';
  zoomBtn.style.backgroundColor = 'white';
  zoomBtn.addEventListener('click', () => ZoomInOut(type));
  return zoomBtn;
};

// Função para lidar com zoom in/out
const ZoomInOut = (type) => {
  const zoomDelta = type === 'in' ? 1 : -1;
  map.setZoom(map.getZoom() + zoomDelta);
};

// Função para formatar a data
const formatDate = (date) => {
  const options = {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    timeZoneName: 'short'
  };
  return new Intl.DateTimeFormat('pt-BR', options).format(date);
};
</script>

<style scoped>
/* Estilos locais para o componente */

.custom-button {
  /* Estilização para o botão de alternar */
  margin-bottom: 64px;
  width: 200px;
  height: 52px;
  padding: 0px 16px 0px 16px;
  border-radius: 6px;
  gap: 16px;
  font-size: 14px;
  font-weight: 500;
  line-height: 36px;
  letter-spacing: 1.25px;
  text-align: center;
  border: none;
  background-position: center;
  transition: background 0.5s;
  background-color: white;
  z-index: 1002;
  position: absolute;
  bottom: 0;
  left: 20px;
}

/* Outros estilos locais... */
</style>

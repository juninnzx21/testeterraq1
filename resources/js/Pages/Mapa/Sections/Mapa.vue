<template>
    <div>
      <div id="map" class="z-[5] h-[calc(100vh)] max-h-[calc(100vh)]"></div>
    </div>
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
  
  import { AdicionaBasemapPadrao } from '@/Pages/Mapa/Functions/TileLayers/AdicionaBasemapPadrao.js';
  import { AdicionaOverlaysPadrao } from '@/Pages/Mapa/Functions/TileLayers/AdicionaOverlaysPadrao.js';
  import { ToggleRasterTile } from '@/Pages/Mapa/Functions/TileLayers/ToggleRasterTile.js';
  
  const props = defineProps({
    projeto: Object,
    rasters: Object
  });
  
  let configsMapa = props.projeto.mapa.configuracao;
  let rasters = props.projeto.rasters;
  let map = ref(null);
  
  onMounted(() => {
    // Objeto com as configurações do Leaflet para criação do objeto map
    let configs = ConfigsLeaflet(configsMapa);
  
    // Cria o objeto map do Leaflet
    map = L.map('map', configs);
  
    // Adiciona a função "SetaVisaoInicial" de resetar a visão inicial do mapa
    FuncaoVisaoInicial(map, configsMapa.visaoInicial.x, configsMapa.visaoInicial.y, configsMapa.visaoInicial.z);
  
    // Seta a visualização inicial do mapa
    SetaVisaoInicial();
  
    // Adiciona as funções de zoom do mapa
    FuncoesZoom(map);
  
    // Adiciona a função "MapaInformacoes" para mostrar as informações do mapa
    FuncaoMapaInformacoes(map);
  
    // Adiciona as coordenadas do mouse no canto inferior esquerdo do mapa
    configsMapa.funcionalidades.coordenadasMouse ? AdicionaCoordenadasMouse(map, configsMapa.configuracoesLeaflet) : configsMapa.funcionalidades.coordenadasMouse;
  
    // Desabilitar o clique-duplo para função de zoom (nativa do Leaflet)
    map.doubleClickZoom.disable();
  
    // Adiciona escala ao mapa caso configsMapa.funcionalidades.escala seja true
    configsMapa.funcionalidades.escala ? AdicionaEscala(map) : configsMapa.funcionalidades.escala;
  
    // Adiciona atribuições (fonte de dados) ao mapa
    configsMapa.funcionalidades.atribuicoes ? AdicionaAttribution(map, configsMapa.configuracoesLeaflet.atribuicaoPrefixo) : map.attributionControl.remove();
  
    // Desativa o clique do botão direito para evitar menu de contexto do navegador e cria o menu de contexto (clique direito com coordenadas)
    configsMapa.funcionalidades.menuContexto ? CriaMenuContexto(map, configsMapa.configuracoesLeaflet) : configsMapa.funcionalidades.menuContexto;
  
    // Adiciona o basemap padrão do mapa (TileLayers)
    AdicionaBasemapPadrao(map, rasters);
  
    // Adiciona os overlays (TileLayers)
    AdicionaOverlaysPadrao(map, rasters);
  
    // Cria a função "ToggleRaster" para alternar a visualização dos overlays
    window.ToggleRaster = function (nomeRaster) {
      rasters.forEach(raster => raster.nome == nomeRaster ? ToggleRasterTile(map, raster) : null);
    };
  
    // Adiciona o botão flutuante de "Zoom In"
const zoomInBtn = L.DomUtil.create('div', 'floating-btn zoom-btn');
zoomInBtn.innerHTML = '<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class=""><circle cx="11" cy="11" r="8"/><line x1="21" x2="16.65" y1="21" y2="16.65"/><line x1="11" x2="11" y1="8" y2="14"/><line x1="8" x2="14" y1="11" y2="11"/></svg>';
zoomInBtn.style.position = 'fixed';
zoomInBtn.style.top = '20px';
zoomInBtn.style.left = '20px';
zoomInBtn.style.zIndex = '1001';
zoomInBtn.addEventListener('click', () => ZoomInOut('in'));
map.getContainer().appendChild(zoomInBtn);

// Adiciona o botão flutuante de "Zoom Out"
const zoomOutBtn = L.DomUtil.create('div', 'floating-btn zoom-btn');
zoomOutBtn.innerHTML = '<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class=""><circle cx="11" cy="11" r="8"/><line x1="21" x2="16.65" y1="21" y2="16.65"/><line x1="8" x2="14" y1="11" y2="11"/></svg>';
zoomOutBtn.style.position = 'fixed';
zoomOutBtn.style.top = '70px';
zoomOutBtn.style.left = '20px';
zoomOutBtn.style.zIndex = '1001';
zoomOutBtn.addEventListener('click', () => ZoomInOut('out'));
map.getContainer().appendChild(zoomOutBtn);
  
    // Altera o estilo do componente de context menu (popup)
    map.on('contextmenu', (e) => {
      const popupContent = `<div>
                            <p>Latitude: ${e.latlng.lat}</p>
                            <p>Longitude: ${e.latlng.lng}</p>
                            <p>Data/Hora: ${formatDate(new Date())}</p>
                            </div>`;
      L.popup().setLatLng(e.latlng).setContent(popupContent).openOn(map);
    });
  });
  
  // Adiciona a função de zoom in/out
  const ZoomInOut = (type) => {
    const zoomDelta = type === 'in' ? 1 : -1;
    map.setZoom(map.getZoom() + zoomDelta);
  };
  
  // Função para formatar a data no formato desejado
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
  /* Adiciona estilos para os botões de zoom */
  .zoom-btn {
    position: fixed;
    top: 20px; /* Ajuste conforme necessário */
    left: 20px; /* Ajuste conforme necessário */
    z-index: 1001; /* Valor maior do que o z-index do mapa */
  }
  
  /* Adiciona estilos para as coordenadas do mouse */
  .leaflet-control-mouseposition {
    position: fixed;
    bottom: 10px;
    left: 10px;
    font-size: 16px;
    background-color: rgba(255, 255, 255, 0.5);
    padding: 5px;
    border-radius: 5px;
    z-index: 1000;
  }
  
  /* Adiciona estilos para o botão flutuante */
  .floating-btn {
    cursor: pointer;
    background-color: #fff;
    border: 1px solid #ddd;
    border-radius: 5px;
    padding: 8px;
    display: flex;
    align-items: center;
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
  }
  
  .floating-btn i {
    font-size: 24px;
  }
  
  /* Adiciona estilos para o contexto do menu (popup) */
  .leaflet-popup-content-wrapper {
    padding: 15px;
  }
  
  .leaflet-popup-content {
    text-align: left;
  }
  </style>
  
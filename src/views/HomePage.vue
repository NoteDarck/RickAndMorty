<template>
  <ion-page>
    <!-- MENU HAMBÚRGUER -->
    <ion-menu content-id="main-content" menu-id="main-menu">
      <ion-header>
        <ion-toolbar style="--background: rgba(66, 0, 219, 0.9);">
          <ion-title>Menu</ion-title>
        </ion-toolbar>
      </ion-header>

      <ion-content class="menu-content">
        <div class="menu-background"></div>
        <ion-list>
          <ion-menu-toggle :auto-hide="false">
            <ion-item button @click="navegarPara('homepage')" class="menu-item">
              <ion-icon :icon="homeOutline" slot="start"></ion-icon>
              Personagens
            </ion-item>
            <ion-item button @click="navegarPara('favoritos')" class="menu-item" :data-count="favoritos.length">
              <ion-icon :icon="heartOutline" slot="start"></ion-icon>
              Favoritos
              <ion-badge slot="end" color="danger" v-if="favoritos.length > 0">{{ favoritos.length }}</ion-badge>
            </ion-item>
            <ion-item button @click="navegarPara('sobre')" class="menu-item">
              <ion-icon :icon="informationCircleOutline" slot="start"></ion-icon>
              Sobre
            </ion-item>
          </ion-menu-toggle>
        </ion-list>
      </ion-content>
    </ion-menu>

    <!-- CONTEÚDO PRINCIPAL (fora do menu) -->
    <div class="main-content-wrapper">
      <!-- HEADER -->
      <ion-header>
        <ion-toolbar style="--background: #4200db;">
          <ion-buttons slot="start">
            <ion-menu-button menu="main-menu"></ion-menu-button>
          </ion-buttons>

          <ion-title>Rick and Morty</ion-title>
        </ion-toolbar>

        <ion-toolbar style="--background: #4200db;">
          <ion-searchbar
            placeholder="Pesquisar personagem"
            @ionInput="buscarPersonagens"
            :debounce="500"
            clear-input
          />
        </ion-toolbar>
      </ion-header>

      <!-- CONTEÚDO -->
      <ion-content id="main-content">
        <!-- Loading State -->
        <div v-if="carregando" class="loading-container">
          <ion-spinner name="crescent" style="color: #9eff00;"></ion-spinner>
          <p>Carregando personagens...</p>
        </div>

        <!-- Mensagem de erro -->
        <div v-else-if="erro" class="error-container">
          <ion-icon :icon="alertCircleOutline" class="error-icon"></ion-icon>
          <h3>Erro ao carregar personagens</h3>
          <p>{{ erro }}</p>
          <ion-button style="--background: #4200db;" @click="carregarPersonagens()">Tentar novamente</ion-button>
        </div>

        <!-- Lista vazia -->
        <div v-else-if="personagens.length === 0 && !carregando" class="empty-container">
          <ion-icon :icon="searchOutline" class="empty-icon"></ion-icon>
          <h3>Nenhum personagem encontrado</h3>
          <p>Tente pesquisar com outro nome</p>
        </div>

        <!-- Grid de personagens -->
        <ion-grid v-else>
          <ion-row>
            <ion-col
              size="12"
              size-sm="6"
              size-md="4"
              size-lg="3"
              v-for="p in personagens"
              :key="p.id"
            >
              <ion-card @click="verDetalhes(p)" @dblclick.stop="toggleFavoritoComDuploClique(p)">
                <div class="image-container">
                  <img :src="p.image" :alt="p.name" loading="lazy" />
                  <ion-badge 
                    class="status-badge"
                    :class="{
                      'alive': p.status === 'Alive',
                      'dead': p.status === 'Dead',
                      'unknown': p.status === 'unknown'
                    }"
                  >
                    {{ p.status }}
                  </ion-badge>
                </div>
                <ion-card-header>
                  <ion-card-title>{{ p.name }}</ion-card-title>
                  <ion-card-subtitle>
                    <ion-icon :icon="planetOutline"></ion-icon>
                    {{ p.species }}
                  </ion-card-subtitle>
                </ion-card-header>
                <ion-card-content>
                  <div class="info-item">
                    <ion-icon :icon="locationOutline"></ion-icon>
                    <span>Origem: {{ p.origin.name }}</span>
                  </div>
                  <div class="info-item">
                    <ion-icon :icon="navigateOutline"></ion-icon>
                    <span>Localização: {{ p.location.name }}</span>
                  </div>
                  <div v-if="p.episode" class="info-item">
                    <ion-icon :icon="filmOutline"></ion-icon>
                    <span>Episódios: {{ p.episode.length }}</span>
                  </div>
                  
                  <!-- Botão de favorito grande -->
                  <div class="favorite-button-container">
                    <ion-button 
                      expand="block" 
                      fill="solid" 
                      :color="p.favorito ? 'danger' : 'primary'"
                      @click.stop="toggleFavorito(p)"
                      class="favorite-main-button"
                    >
                      <ion-icon 
                        slot="start" 
                        :icon="p.favorito ? heart : heartOutline"
                      ></ion-icon>
                      {{ p.favorito ? 'REMOVER DOS FAVORITOS' : 'ADICIONAR AOS FAVORITOS' }}
                    </ion-button>
                  </div>
                </ion-card-content>
                
                <!-- Botão de coração rápido no canto -->
                <ion-fab vertical="top" horizontal="end" slot="fixed">
                  <ion-fab-button size="small" color="danger" @click.stop="toggleFavorito(p)">
                    <ion-icon :icon="p.favorito ? heart : heartOutline"></ion-icon>
                  </ion-fab-button>
                </ion-fab>
              </ion-card>
            </ion-col>
          </ion-row>
        </ion-grid>

        <!-- Botão para carregar mais -->
        <div v-if="info && info.next && !carregando" class="load-more-container">
          <ion-button 
            expand="block" 
            fill="outline" 
            style="--color: #4200db; --border-color: #4200db;"
            @click="carregarMais"
            :disabled="carregandoMais"
          >
            <ion-spinner v-if="carregandoMais" name="crescent"></ion-spinner>
            <span v-else>Carregar mais personagens</span>
          </ion-button>
        </div>
      </ion-content>

      <!-- Toast para feedback -->
      <ion-toast
        :is-open="toast.visible"
        :message="toast.message"
        :duration="2000"
        @didDismiss="toast.visible = false"
      ></ion-toast>
    </div>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonSearchbar,
  IonGrid,
  IonRow,
  IonCol,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardSubtitle,
  IonCardContent,
  IonMenu,
  IonList,
  IonItem,
  IonMenuButton,
  IonButtons,
  IonMenuToggle,
  IonIcon,
  IonSpinner,
  IonButton,
  IonBadge,
  IonFab,
  IonFabButton,
  IonToast,
  onIonViewWillEnter
} from '@ionic/vue'

import { ref, onMounted } from 'vue'
import {
  homeOutline,
  heartOutline,
  heart,
  informationCircleOutline,
  alertCircleOutline,
  searchOutline,
  planetOutline,
  locationOutline,
  navigateOutline,
  filmOutline,
} from 'ionicons/icons'

// Interface para tipagem
interface Personagem {
  id: number
  name: string
  species: string
  status: 'Alive' | 'Dead' | 'unknown'
  type: string
  gender: string
  origin: {
    name: string
    url: string
  }
  location: {
    name: string
    url: string
  }
  image: string
  episode: string[]
  url: string
  created: string
  favorito?: boolean
}

interface InfoAPI {
  count: number
  pages: number
  next: string | null
  prev: string | null
}

const personagens = ref<Personagem[]>([])
const info = ref<InfoAPI | null>(null)
const carregando = ref(false)
const carregandoMais = ref(false)
const erro = ref<string | null>(null)
const favoritos = ref<number[]>([])
const toast = ref({
  visible: false,
  message: ''
})

// GET da API
const carregarPersonagens = async (nome = '', url?: string) => {
  try {
    if (!url) {
      carregando.value = true
      personagens.value = []
    } else {
      carregandoMais.value = true
    }
    
    erro.value = null
    
    let apiUrl = url || 'https://rickandmortyapi.com/api/character'
    
    if (nome.trim() && !url) {
      apiUrl += `?name=${encodeURIComponent(nome.trim())}`
    }

    const response = await fetch(apiUrl)
    
    if (!response.ok) {
      if (response.status === 404) {
        personagens.value = []
        info.value = null
        return
      }
      throw new Error(`Erro ${response.status}: ${response.statusText}`)
    }
    
    const data = await response.json()
    
    // Marcar favoritos
    const personagensComFavoritos = data.results.map((p: Personagem) => ({
      ...p,
      favorito: favoritos.value.includes(p.id)
    }))
    
    if (url) {
      // Adicionar à lista existente
      personagens.value = [...personagens.value, ...personagensComFavoritos]
    } else {
      // Nova lista
      personagens.value = personagensComFavoritos
    }
    
    info.value = data.info
    
  } catch (error: any) {
    console.error('Erro ao carregar personagens:', error)
    erro.value = error.message || 'Erro desconhecido'
    
    if (!url) {
      personagens.value = []
    }
  } finally {
    carregando.value = false
    carregandoMais.value = false
  }
}

// Debounce para evitar muitas requisições
let timeoutId: ReturnType<typeof setTimeout> | null = null

const buscarPersonagens = (event: CustomEvent) => {
  if (timeoutId) clearTimeout(timeoutId)
  
  const valor = event.detail.value || ''
  
  timeoutId = setTimeout(() => {
    carregarPersonagens(valor)
  }, 500)
}

// Carregar mais personagens
const carregarMais = () => {
  if (info.value?.next) {
    carregarPersonagens('', info.value.next)
  }
}

// Toggle favorito
const toggleFavorito = (personagem: Personagem) => {
  const index = favoritos.value.indexOf(personagem.id)
  
  if (index > -1) {
    // Remover dos favoritos
    favoritos.value.splice(index, 1)
    personagem.favorito = false
    toast.value = {
      visible: true,
      message: `❌ ${personagem.name} removido dos favoritos`
    }
  } else {
    // Adicionar aos favoritos
    favoritos.value.push(personagem.id)
    personagem.favorito = true
    toast.value = {
      visible: true,
      message: `❤️ ${personagem.name} adicionado aos favoritos`
    }
  }
  
  // Salvar no localStorage
  localStorage.setItem('rickandmorty_favoritos', JSON.stringify(favoritos.value))
}

// Toggle favorito com duplo clique
const toggleFavoritoComDuploClique = (personagem: Personagem) => {
  toggleFavorito(personagem)
  // Feedback visual extra para duplo clique
  const card = event?.target as HTMLElement
  if (card) {
    card.style.transform = 'scale(0.98)'
    setTimeout(() => {
      card.style.transform = ''
    }, 200)
  }
}

// Ver detalhes do personagem
const verDetalhes = (personagem: Personagem) => {
  // Aqui você pode implementar navegação para página de detalhes
  console.log('Ver detalhes do personagem:', personagem)
  // Exemplo: router.push(`/personagem/${personagem.id}`)
}

// Navegação do menu
const navegarPara = (destino: string) => {
  console.log('Navegar para:', destino)
  
  if (destino === 'homepage') {
    // Homepage mostra todos os personagens
    carregarPersonagens()
  } else if (destino === 'favoritos') {
    // Verificar se há favoritos salvos
    if (favoritos.value.length === 0) {
      toast.value = {
        visible: true,
        message: 'Você não tem personagens favoritos ainda! ❤️'
      }
      return
    }
    
    // Se já temos personagens carregados, filtrar
    if (personagens.value.length > 0) {
      const favoritosAtual = personagens.value.filter(p => p.favorito === true)
      if (favoritosAtual.length === 0) {
        // Pode acontecer se os favoritos são de outra sessão
        // Nesse caso, precisamos recarregar a API filtrando por IDs
        const idsFavoritos = favoritos.value.join(',')
        carregarFavoritosDaAPI(idsFavoritos)
      } else {
        // Criar uma cópia dos favoritos
        const favoritosFiltrados = [...favoritosAtual]
        personagens.value = favoritosFiltrados
        info.value = null // Remove paginação quando em favoritos
        toast.value = {
          visible: true,
          message: `Mostrando ${favoritosFiltrados.length} favorito(s) ❤️`
        }
      }
    } else {
      // Se não temos personagens carregados, carregar apenas os favoritos
      const idsFavoritos = favoritos.value.join(',')
      carregarFavoritosDaAPI(idsFavoritos)
    }
  } else if (destino === 'sobre') {
    // Navegar para página sobre
    alert('Sobre o App Rick and Morty\n\nDesenvolvido com Ionic Vue\n\nEste aplicativo consome a API pública do Rick and Morty para mostrar informações sobre os personagens da série.\n\n✨ Funcionalidades:\n• Lista de personagens\n• Sistema de favoritos\n• Busca por nome\n• Filtro por status\n• Detalhes dos personagens')
  }
}

// Função para carregar favoritos diretamente da API
const carregarFavoritosDaAPI = async (ids: string) => {
  if (!ids) return
  
  carregando.value = true
  try {
    const response = await fetch(`https://rickandmortyapi.com/api/character/${ids}`)
    if (!response.ok) throw new Error('Erro ao carregar favoritos')
    
    const data = await response.json()
    // A API retorna array se múltiplos IDs, ou objeto se único
    const resultados = Array.isArray(data) ? data : [data]
    
    const personagensComFavoritos = resultados.map((p: Personagem) => ({
      ...p,
      favorito: true
    }))
    
    personagens.value = personagensComFavoritos
    info.value = null // Remove paginação
    
    toast.value = {
      visible: true,
      message: `Carregados ${personagensComFavoritos.length} favorito(s) ❤️`
    }
  } catch (error) {
    console.error('Erro ao carregar favoritos:', error)
    toast.value = {
      visible: true,
      message: 'Erro ao carregar personagens favoritos ❌'
    }
  } finally {
    carregando.value = false
  }
}

// Carregar favoritos do localStorage
const carregarFavoritos = () => {
  const salvos = localStorage.getItem('rickandmorty_favoritos')
  if (salvos) {
    favoritos.value = JSON.parse(salvos)
  }
}

onMounted(() => {
  carregarFavoritos()
  carregarPersonagens()
})

// Recarregar quando a página for exibida novamente
onIonViewWillEnter(() => {
  // Atualizar favoritos se necessário
  carregarFavoritos()
})
</script>

<style scoped>
/* Ajuste do wrapper principal */
.main-content-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
}

/* Fundo do Rick and Morty - APENAS no conteúdo principal */
#main-content {
  --background: url('./imgs/Fundo.jpg') no-repeat center center / cover;
  min-height: 100vh;
}

/* Menu com imagem de fundo */
.menu-content {
  --background: transparent;
  position: relative;
  overflow: hidden;
}

.menu-background {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: url('./imgs/Fundo.jpg') no-repeat center center / cover;
  filter: brightness(0.6) contrast(1.1);
  z-index: -1;
}

.menu-background::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    135deg,
    rgba(66, 0, 219, 0.8) 0%,
    rgba(158, 255, 0, 0.4) 100%
  );
}

/* Items do menu */
ion-list {
  background: transparent;
  padding: 20px 0;
  margin-top: 20px;
}

.menu-item {
  --background: rgba(255, 255, 255, 0.15);
  --color: white;
  --padding-start: 25px;
  --padding-end: 25px;
  --min-height: 60px;
  --border-radius: 12px;
  --detail-icon-color: #9eff00;
  --detail-icon-opacity: 1;
  margin: 12px 20px;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.menu-item::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.2),
    transparent
  );
  transition: left 0.5s ease;
}

.menu-item:hover::before {
  left: 100%;
}

.menu-item:hover {
  --background: rgba(158, 255, 0, 0.25);
  transform: translateX(8px);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(158, 255, 0, 0.5);
}

.menu-item ion-icon {
  color: #9eff00;
  font-size: 22px;
  margin-right: 15px;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.3));
}

/* Badge de contador no menu */
ion-badge {
  font-weight: bold;
  font-size: 12px;
  min-width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Containers de estado */
.loading-container,
.error-container,
.empty-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 60vh;
  text-align: center;
  padding: 20px;
  color: white;
  background: rgba(0, 0, 0, 0.7);
  border-radius: 10px;
  margin: 20px;
  backdrop-filter: blur(5px);
}

.loading-container p,
.error-container h3,
.error-container p,
.empty-container h3,
.empty-container p {
  color: white;
  margin-top: 10px;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.8);
}

.error-icon,
.empty-icon {
  font-size: 64px;
  margin-bottom: 20px;
  color: #ff6b6b;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.5));
}

.empty-icon {
  color: #9eff00;
}

/* Cards */
ion-card {
  background: rgba(0, 0, 0, 0.85);
  color: #ffffff;
  border-radius: 15px;
  overflow: hidden;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  height: 100%;
  margin: 0;
  cursor: pointer;
  position: relative;
  border: 1px solid rgba(158, 255, 0, 0.2);
}

ion-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(66, 0, 219, 0.4);
}

/* Indicador de duplo clique */
ion-card::after {
  content: 'Duplo clique para favoritar rápido!';
  position: absolute;
  bottom: 5px;
  right: 5px;
  font-size: 9px;
  color: rgba(158, 255, 0, 0.9);
  background: rgba(0, 0, 0, 0.9);
  padding: 3px 6px;
  border-radius: 4px;
  opacity: 0;
  transition: opacity 0.3s;
  z-index: 10;
  border: 1px solid rgba(158, 255, 0, 0.3);
}

ion-card:hover::after {
  opacity: 1;
}

.image-container {
  position: relative;
  width: 100%;
  height: 250px;
  overflow: hidden;
}

.image-container img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

ion-card:hover .image-container img {
  transform: scale(1.05);
}

.status-badge {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 12px;
  padding: 5px 10px;
  border-radius: 12px;
  z-index: 2;
  font-weight: bold;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.status-badge.alive {
  background: linear-gradient(135deg, #10dc60, #0cb850);
  color: #000;
  border: 1px solid rgba(0, 0, 0, 0.2);
}

.status-badge.dead {
  background: linear-gradient(135deg, #f04141, #d32f2f);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.status-badge.unknown {
  background: linear-gradient(135deg, #6a6a6a, #4a4a4a);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

ion-card-header {
  padding: 16px 16px 8px;
}

ion-card-title {
  font-size: 1.1rem;
  font-weight: bold;
  color: #ffffff;
  margin-bottom: 4px;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.5);
}

ion-card-subtitle {
  color: #9eff00;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 4px;
  font-weight: 600;
}

ion-card-content {
  padding: 0 16px 16px;
}

.info-item {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 12px;
  font-size: 0.85rem;
  color: #cccccc;
  padding: 8px 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.info-item:last-child {
  margin-bottom: 16px;
  border-bottom: none;
}

.info-item ion-icon {
  color: #9eff00;
  font-size: 16px;
  flex-shrink: 0;
  min-width: 20px;
}

/* Botão de favorito principal */
.favorite-button-container {
  margin-top: 16px;
  padding-top: 16px;
  border-top: 2px solid rgba(158, 255, 0, 0.3);
}

.favorite-main-button {
  --background: linear-gradient(135deg, #4200db, #5200ff);
  --background-hover: linear-gradient(135deg, #5200ff, #6200ee);
  --background-activated: linear-gradient(135deg, #3200b8, #4200db);
  --color: white;
  font-weight: bold;
  font-size: 0.85rem;
  --border-radius: 10px;
  margin: 0;
  height: 44px;
  --box-shadow: 0 4px 8px rgba(66, 0, 219, 0.3);
}

.favorite-main-button[color="danger"] {
  --background: linear-gradient(135deg, #f04141, #ff5252);
  --background-hover: linear-gradient(135deg, #ff5252, #ff6b6b);
  --background-activated: linear-gradient(135deg, #d32f2f, #f04141);
  --box-shadow: 0 4px 8px rgba(240, 65, 65, 0.3);
}

/* Botão flutuante de coração rápido */
ion-fab {
  z-index: 10;
}

ion-fab-button {
  --background: linear-gradient(135deg, #f04141, #ff5252);
  --background-hover: linear-gradient(135deg, #ff5252, #ff6b6b);
  --background-activated: linear-gradient(135deg, #d32f2f, #f04141);
  --box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
  --color: white;
}

/* Header e título */
ion-toolbar {
  --background: #4200db;
  --color: white;
}

ion-title {
  font-size: 1.5rem;
  font-weight: bold;
  text-align: center;
  font-family: 'Arial Rounded MT Bold', 'Arial', sans-serif;
  letter-spacing: 1px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
}

/* Searchbar personalizado */
ion-searchbar {
  --background: rgba(255, 255, 255, 0.95);
  --color: #000000;
  --placeholder-color: #666666;
  --icon-color: #4200db;
  --border-radius: 50px;
  --box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

/* Menu principal */
ion-menu {
  --width: 300px;
  --background: transparent;
}

ion-menu ion-toolbar {
  --background: rgba(66, 0, 219, 0.95);
  backdrop-filter: blur(10px);
}

/* Botão carregar mais */
.load-more-container {
  padding: 20px;
  max-width: 300px;
  margin: 30px auto;
}

/* Grid responsivo */
ion-col {
  padding: 10px;
  animation: fadeIn 0.5s ease-out;
}

/* Animações */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.05); }
  100% { transform: scale(1); }
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(-20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.pulse-animation {
  animation: pulse 0.5s ease;
}

.menu-item {
  animation: slideIn 0.3s ease-out;
}

.menu-item:nth-child(1) { animation-delay: 0.1s; }
.menu-item:nth-child(2) { animation-delay: 0.2s; }
.menu-item:nth-child(3) { animation-delay: 0.3s; }

/* Responsividade */
@media (max-width: 576px) {
  .image-container {
    height: 200px;
  }
  
  ion-title {
    font-size: 1.2rem;
  }
  
  ion-card::after {
    font-size: 8px;
    padding: 2px 4px;
  }
  
  .favorite-main-button {
    font-size: 0.75rem;
    height: 40px;
  }
  
  .menu-item {
    --padding-start: 20px;
    --padding-end: 20px;
    margin: 10px 15px;
  }
  
  ion-menu {
    --width: 280px;
  }
}

@media (min-width: 768px) {
  .image-container {
    height: 280px;
  }
  
  ion-menu {
    --width: 320px;
  }
}

@media (min-width: 992px) {
  .menu-background {
    background-attachment: fixed;
  }
}

/* Ajuste para garantir que o conteúdo principal fique atrás do menu quando aberto */
ion-menu[open] ~ .main-content-wrapper {
  transform: translateX(300px);
  transition: transform 0.3s ease;
}

@media (max-width: 576px) {
  ion-menu[open] ~ .main-content-wrapper {
    transform: translateX(280px);
  }
}
</style>
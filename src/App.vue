<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'

type Movie = { id: number; title: string; year: string; rating: number; genres: string[]; duration: string; poster: string; backdrop: string; overview: string; featured?: boolean }
const apiKey = import.meta.env.VITE_TMDB_API_KEY
const imageBase = 'https://image.tmdb.org/t/p/w780'
const activeGenre = ref('Todos')
const search = ref('')
const sortBy = ref('Mais relevantes')
const activeTab = ref('explorar')
const selectedMovie = ref<Movie | null>(null)
const favorites = ref<number[]>([])
const isLoading = ref(false)
const usingApi = ref(false)
const favoritesStorageKey = 'cinevault:favorites'

const movies = ref<Movie[]>([
  { id: 872585, title: 'Oppenheimer', year: '2023', rating: 8.1, genres: ['Drama', 'História'], duration: '3h 01min', poster: 'https://image.tmdb.org/t/p/w780/8Gxv8gSFCU0XGDykEGv7zR1n2ua.jpg', backdrop: 'https://image.tmdb.org/t/p/w1280/fm6KqXpk3M2HVveHwCrBSSBaO0V.jpg', overview: 'A história do físico americano J. Robert Oppenheimer e seu papel no desenvolvimento da bomba atômica.', featured: true },
  { id: 693134, title: 'Duna: Parte Dois', year: '2024', rating: 8.2, genres: ['Ficção', 'Aventura'], duration: '2h 46min', poster: 'https://image.tmdb.org/t/p/w780/1pdfLvkbY9ohJlCjQH2CZjjYVvJ.jpg', backdrop: 'https://image.tmdb.org/t/p/w1280/xOMo8BRK7PfcJv9JCnx7s5hj0PX.jpg', overview: 'Paul Atreides se une a Chani e aos Fremen em uma jornada de vingança contra os conspiradores que destruíram sua família.', featured: true },
  { id: 872906, title: 'Pobres Criaturas', year: '2023', rating: 7.7, genres: ['Drama', 'Romance'], duration: '2h 21min', poster: 'https://image.tmdb.org/t/p/w780/kCGlIMHnOm8JPXq3rXM6c5wMxcT.jpg', backdrop: 'https://image.tmdb.org/t/p/w1280/jmQdJ3kR3E3r9cN6D3q9G5J8YvV.jpg', overview: 'A incrível história e a fantástica evolução de Bella Baxter, uma jovem trazida de volta à vida por um cientista.' },
  { id: 872876, title: 'Assassinos da Lua das Flores', year: '2023', rating: 7.5, genres: ['Crime', 'Drama'], duration: '3h 26min', poster: 'https://image.tmdb.org/t/p/w780/dB6Krk806zeqd0YNp2ngQ9zXteH.jpg', backdrop: 'https://image.tmdb.org/t/p/w1280/3qW7Y8iFfP2zP4HnS9m7dC6bF8a.jpg', overview: 'Na década de 1920, membros da nação Osage são assassinados um a um, provocando uma investigação extraordinária.' },
  { id: 447365, title: 'Guardiões da Galáxia Vol. 3', year: '2023', rating: 7.9, genres: ['Ação', 'Ficção'], duration: '2h 30min', poster: 'https://image.tmdb.org/t/p/w780/r2J02Z2OpNTctfOSN1Ydgii51I3.jpg', backdrop: 'https://image.tmdb.org/t/p/w1280/5YZbUmjbMa3ClvSW1Wj3D6XGolb.jpg', overview: 'Peter Quill ainda está abalado pela perda de Gamora e precisa reunir sua equipe para defender o universo.' },
  { id: 346698, title: 'Barbie', year: '2023', rating: 7.0, genres: ['Comédia', 'Aventura'], duration: '1h 54min', poster: 'https://image.tmdb.org/t/p/w780/iuFNMS8U5cb6xfzi51Dbkovj7vM.jpg', backdrop: 'https://image.tmdb.org/t/p/w1280/ctMserH8g2SeOAnCw5gFjdQF8MO.jpg', overview: 'Viver na Barbielândia é ser um ser perfeito em um lugar perfeito. A menos que você esteja passando por uma crise existencial.' },
  { id: 569094, title: 'Homem-Aranha: Através do Aranhaverso', year: '2023', rating: 8.4, genres: ['Animação', 'Ação'], duration: '2h 20min', poster: 'https://image.tmdb.org/t/p/w780/8Vt6mWEReuy4Of61Lnj5Xj704m8.jpg', backdrop: 'https://image.tmdb.org/t/p/w1280/4HodYYKEIsGOdinkGi2Ucz6X9i0.jpg', overview: 'Miles Morales é catapultado pelo multiverso e encontra uma equipe de Pessoas-Aranha encarregada de proteger sua existência.' },
  { id: 76600, title: 'Avatar: O Caminho da Água', year: '2022', rating: 7.6, genres: ['Ficção', 'Aventura'], duration: '3h 12min', poster: 'https://image.tmdb.org/t/p/w780/t6HIqrRAclMCA60NsSmeqe9RmNV.jpg', backdrop: 'https://image.tmdb.org/t/p/w1280/s16H6tpK2utvwDtzZ8Qy4qm5Emw.jpg', overview: 'Mais de uma década depois dos acontecimentos do primeiro filme, a família Sully enfrenta uma nova ameaça.' },
])
const genres = computed(() => ['Todos', ...new Set(movies.value.flatMap((movie) => movie.genres))])
const featured = computed<Movie>(() => movies.value.find((movie) => movie.featured) ?? movies.value[0]!)
const filteredMovies = computed(() => {
  let result = movies.value.filter((movie) => (activeGenre.value === 'Todos' || movie.genres.includes(activeGenre.value)) && movie.title.toLowerCase().includes(search.value.toLowerCase()))
  if (sortBy.value === 'Melhor avaliados') result = [...result].sort((a, b) => b.rating - a.rating)
  if (sortBy.value === 'Mais recentes') result = [...result].sort((a, b) => Number(b.year) - Number(a.year))
  return result
})
function toggleFavorite(id: number) {
  favorites.value = favorites.value.includes(id) ? favorites.value.filter((favorite) => favorite !== id) : [...favorites.value, id]
  localStorage.setItem(favoritesStorageKey, JSON.stringify(favorites.value))
}
function isFavorite(id: number) { return favorites.value.includes(id) }
function clearSearch() { search.value = ''; activeGenre.value = 'Todos' }
function openTrailer(movie: Movie) {
  const query = `${movie.title} trailer oficial`
  window.open(`https://www.youtube.com/results?search_query=${encodeURIComponent(query)}`, '_blank', 'noopener,noreferrer')
}
function handleImageError(event: Event) {
  const image = event.target as HTMLImageElement
  image.src = 'https://image.tmdb.org/t/p/w780/1pdfLvkbY9ohJlCjQH2CZjjYVvJ.jpg'
}
async function fetchMovies() {
  if (!apiKey) return
  isLoading.value = true
  try {
    const response = await fetch(`https://api.themoviedb.org/3/movie/popular?api_key=${apiKey}&language=pt-BR&page=1`)
    if (!response.ok) throw new Error('TMDB request failed')
    const data = await response.json()
    movies.value = data.results.filter((item: Record<string, unknown>) => item.poster_path).map((item: Record<string, unknown>) => ({ id: item.id as number, title: item.title as string, year: String(item.release_date).slice(0, 4), rating: Number((item.vote_average as number).toFixed(1)), genres: ['Cinema'], duration: '—', poster: `${imageBase}${item.poster_path}`, backdrop: `${imageBase}${item.backdrop_path ?? item.poster_path}`, overview: (item.overview as string) || 'Sem sinopse disponível.' }))
    usingApi.value = true
  } catch (error) { console.warn('Catálogo local exibido:', error) } finally { isLoading.value = false }
}
onMounted(() => {
  const savedFavorites = localStorage.getItem(favoritesStorageKey)
  if (savedFavorites) {
    try {
      favorites.value = JSON.parse(savedFavorites) as number[]
    } catch {
      localStorage.removeItem(favoritesStorageKey)
    }
  }
  fetchMovies()
})
</script>

<template>
  <div class="app-shell">
    <header class="topbar"><a class="brand" href="#"><span class="brand-mark">◒</span><span>cine<span>vault</span></span></a><nav class="main-nav"><button :class="{ active: activeTab === 'explorar' }" @click="activeTab = 'explorar'">Explorar</button><button :class="{ active: activeTab === 'minha-lista' }" @click="activeTab = 'minha-lista'">Minha lista <b v-if="favorites.length">{{ favorites.length }}</b></button></nav><div class="top-actions"><button class="avatar">AF</button></div></header>
    <main>
      <section v-if="activeTab === 'explorar'" class="hero" :style="{ backgroundImage: `linear-gradient(90deg, #101414 0%, rgba(16,20,20,.92) 32%, rgba(16,20,20,.2) 78%), url(${featured.backdrop})` }"><div class="hero-copy"><span class="eyebrow"><i></i> Destaque da semana</span><h1>{{ featured.title }}</h1><div class="hero-meta"><span class="rating">★ {{ featured.rating }}</span><span>{{ featured.year }}</span><span>{{ featured.duration }}</span><span v-for="genre in featured.genres" :key="genre">{{ genre }}</span></div><p>{{ featured.overview }}</p><div class="hero-actions"><button class="primary-button" @click="openTrailer(featured)">▶ Assistir trailer</button><button class="ghost-button" @click="toggleFavorite(featured.id)">{{ isFavorite(featured.id) ? '♥ Na sua lista' : '+ Minha lista' }}</button></div></div><div class="hero-scroll">role para explorar <span>↓</span></div></section>
      <section v-if="activeTab === 'explorar'" class="catalog section-wrap"><div class="section-heading"><div><span class="eyebrow muted">Descubra algo novo</span><h2>Em cartaz agora</h2></div><label class="search-box"><span>⌕</span><input v-model="search" placeholder="Buscar filmes..." /><button v-if="search" @click="search = ''">×</button></label></div><div class="filter-row"><div class="genre-filters"><button v-for="genre in genres" :key="genre" :class="{ selected: activeGenre === genre }" @click="activeGenre = genre">{{ genre }}</button></div><select v-model="sortBy" aria-label="Ordenar filmes"><option>Mais relevantes</option><option>Melhor avaliados</option><option>Mais recentes</option></select></div><div v-if="isLoading" class="status-message">Carregando novidades do cinema...</div><div v-else-if="filteredMovies.length" class="movie-grid"><article v-for="movie in filteredMovies" :key="movie.id" class="movie-card" @click="selectedMovie = movie"><div class="poster-wrap"><img :src="movie.poster" :alt="`Pôster de ${movie.title}`" loading="lazy" @error="handleImageError" /><span class="card-rating">★ {{ movie.rating }}</span><button class="favorite-button" :class="{ saved: isFavorite(movie.id) }" @click.stop="toggleFavorite(movie.id)">{{ isFavorite(movie.id) ? '♥' : '♡' }}</button><div class="play-overlay">▶</div></div><div class="movie-info"><h3>{{ movie.title }}</h3><p>{{ movie.year }} <span>•</span> {{ movie.genres.join(' · ') }}</p></div></article></div><div v-else class="empty-state"><span>⌕</span><h3>Nenhum filme encontrado</h3><p>Tente buscar outro título ou remover os filtros.</p><button class="ghost-button" @click="clearSearch">Limpar filtros</button></div><div class="catalog-footer"><span>{{ filteredMovies.length }} títulos disponíveis</span><span class="api-status"><i :class="{ live: usingApi }"></i>{{ usingApi ? 'Sincronizado com TMDB' : 'Catálogo editorial' }}</span></div></section>
      <section v-else class="section-wrap list-view"><span class="eyebrow muted">Seu cinema pessoal</span><h2>Minha lista</h2><div v-if="favorites.length" class="movie-grid"><article v-for="movie in movies.filter((item) => favorites.includes(item.id))" :key="movie.id" class="movie-card" @click="selectedMovie = movie"><div class="poster-wrap"><img :src="movie.poster" :alt="`Pôster de ${movie.title}`" @error="handleImageError" /><button class="favorite-button saved" @click.stop="toggleFavorite(movie.id)">♥</button></div><div class="movie-info"><h3>{{ movie.title }}</h3><p>{{ movie.year }} <span>•</span> {{ movie.genres.join(' · ') }}</p></div></article></div><div v-else class="empty-state"><span>♡</span><h3>Sua lista está vazia</h3><p>Salve filmes para encontrá-los aqui depois.</p><button class="primary-button" @click="activeTab = 'explorar'">Explorar catálogo</button></div></section>
    </main>
    <div v-if="selectedMovie" class="modal-backdrop" @click.self="selectedMovie = null"><article class="detail-modal"><button class="modal-close" aria-label="Fechar" @click="selectedMovie = null">×</button><img class="modal-image" :src="selectedMovie.backdrop" :alt="selectedMovie.title" @error="handleImageError" /><div class="modal-content"><span class="eyebrow">Detalhes do filme</span><h2>{{ selectedMovie.title }}</h2><div class="hero-meta"><span class="rating">★ {{ selectedMovie.rating }}</span><span>{{ selectedMovie.year }}</span><span>{{ selectedMovie.duration }}</span></div><p>{{ selectedMovie.overview }}</p><div class="hero-actions"><button class="primary-button" @click="openTrailer(selectedMovie)">▶ Ver trailer no YouTube</button><button class="ghost-button" @click="toggleFavorite(selectedMovie.id)">{{ isFavorite(selectedMovie.id) ? '♥ Remover da lista' : '+ Adicionar à lista' }}</button></div></div></article></div>
  </div>
</template>

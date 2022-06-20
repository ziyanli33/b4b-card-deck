<script setup lang="ts">
import {computed, Ref, ref} from 'vue'
import allCards from "../assets/cards.json"
import Index from 'flexsearch'
import CRC32 from 'crc-32'
import * as base62 from 'base62-ts'

interface Card {
  name: string,
  description: string,
  type: string,
  affinity: string
}
let filteredCards: Ref<number[]> = ref(Array.from(Array(allCards.length).keys()))
let searchToken: Ref<string> = ref('')
let remainingDraws = computed(() => 15 - selectedCards.value.length)

let filterTypes = ['Offense', 'Defense', 'Utility', 'Mobility', 'Ability', 'Talent', 'Stat']
let filterTypeChecked = ref([true, true, true, true, true, true, true])
let filterAffinities = ['Reflex', 'Fortune', 'Brawn', 'Discipline']
let filterAffinityChecked = ref([true, true, true, true])

let decoderMap = new Map<string, number>()
let selectedCards: Ref<number[]> = ref([])
let combo = new URLSearchParams(window.location.search).get('combo')

const site = location.protocol + '//' + location.host

// Initialize search engine and decoder map
const index = new Index.Index()
for (let i = 0; i < allCards.length; i++) {
  index.add(i, allCards[i].name + " " + allCards[i].description)
  decoderMap.set(encode(allCards[i].name), i)
}

// Determine initial selected cards from url.
if (combo) {
  let encodedCardNames = combo.split("-")
  for (let i = 0; i < encodedCardNames.length; i++) {
    if (decoderMap.has(encodedCardNames[i])) {
      selectedCards.value.push(Number(decoderMap.get(encodedCardNames[i])))
    }
  }
}

let shareLink = computed(() => {
  return site + "?combo=" + selectedCards.value.map((i) => {
    return encode(getCard(i).name)
  }).join("-")
})

function share() {
  let v = shareLink.value
  // let v = site + "?combo=" + selectedCards.value.map((i) => {
  //   return encode(getCard(i).name)
  // }).join("-")
  navigator.clipboard.writeText(v)
  alert("The following link has been copied to your clipboard: \n\n" + v)
}

function encode(text: string): string {
  return base62.encode(CRC32.str(text) >>> 0)
}

function search() {
  if (searchToken.value == '') {
    filteredCards.value = Array.from(Array(allCards.length).keys())
    return
  }
  filteredCards.value = index.search(searchToken.value).map(Number)
}

function select(i: number) {
  if (!selected(i) && selectedCards.value.length < 15) {
    selectedCards.value.push(i)
  }
}

function getCard(i: number): Card {
  return allCards[i]
}

function selected(i: number): boolean {
  return selectedCards.value.includes(i)
}

function remove(i: number) {
  const index: number = selectedCards.value.indexOf(i);
  if (index > -1) {
    selectedCards.value.splice(index, 1);
  }
}

function filter() {
  let newFilteredCards = []
  for (let i = 0; i < filteredCards.value.length; i++) {
    if(!filterTypeChecked.value.every(value => value == false)){
      if (!filterTypeChecked.value[filterTypes.indexOf(allCards[filteredCards.value[i]].type)]){
        continue
      }
    }
    if(!filterAffinityChecked.value.every(value => value == false)){
      if(!filterAffinityChecked.value[filterAffinities.indexOf(allCards[filteredCards.value[i]].affinity)]){
        continue
      }
    }
    newFilteredCards.push(filteredCards.value[i])
  }
  filteredCards.value = newFilteredCards
}

function apply() {
  search()
  filter()
}

function toSnakeCase(inputString: string) {
  inputString = inputString.replaceAll("!", "").replaceAll("'", "").replaceAll(".", "").replaceAll("-","")
  return inputString.split(' ').map((character) => {
    return character.toLowerCase();
  }).join('_');
}

</script>

<template>
  <div>
    <!-- top navbar -->
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
      <div class="container-fluid">
        <a class="navbar-brand" href="#">Back4Blood Card Deck</a>
        <input class="form-control me-2" type="text" placeholder="Search" aria-label="Search"
               v-model="searchToken" @keydown.enter="apply">
        <div class="collapse navbar-collapse" id="navbarNavDarkDropdown">
          <ul class="navbar-nav">
            <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="#" id="navbarDarkDropdownMenuLink" role="button"
                 data-bs-toggle="dropdown" aria-expanded="false">
                Filter
              </a>
              <ul class="dropdown-menu dropdown-menu-dark" aria-labelledby="navbarDarkDropdownMenuLink">
                <li class="dropdown-item" v-for="(checked, i) in filterTypeChecked">
                  <input type="checkbox" class="form-check-input" :id="filterTypes[i]" v-model="filterTypeChecked[i]">
                  <label class="form-check-label" :for="filterTypes[i]">{{ filterTypes[i] }}</label>
                </li>
                <li>
                  <hr class="dropdown-divider">
                </li>
                <li class="dropdown-item" v-for="(checked, i) in filterAffinityChecked">
                  <input type="checkbox" class="form-check-input" :id="filterAffinities[i]"
                         v-model="filterAffinityChecked[i]">
                  <label class="form-check-label" :for="filterAffinities[i]">{{ filterAffinities[i] }}</label>
                </li>
              </ul>
            </li>
          </ul>
        </div>
        <button type="submit" class="btn btn-primary" @click="apply">Apply</button>
      </div>
    </nav>
  </div>
  <div class="row">
    <div class="col-3">
      <!-- sidebar -->
      <div class="d-flex flex-column flex-shrink-0 p-3 bg-light sticky-top">
        <h2>My Deck</h2>
        <ul class="nav nav-pills flex-column mb-auto">
          <li class="nav-item d-grid" v-for="i in selectedCards" :key="i" @click="remove(i)">
            <button class="btn btn-outline-dark">{{ getCard(i).name }}</button>
          </li>
          <li class="nav-item d-grid" v-for="i in remainingDraws">
            <button class="btn btn-outline-light">empty</button>
          </li>
        </ul>
        <input type="text" v-model="shareLink">
        <button class="btn btn-outline-success" @click="share">Share My Deck</button>

      </div>
    </div>
    <div class="col-9">
      <!-- main content -->
      <div>
        <p>Showing {{ filteredCards.length }} cards</p>
      </div>
      <div class="container" style="margin-top: 20px">
        <div class="row row-cols-1 row-cols-sm-2 row-cols-md-3 row-cols-lg-4 row-cols-xl-5">
          <div class="col mb-4" v-for="i in filteredCards" :key="i" @click="select(i)">
            <div class="card" :class="{ 'bg-light border-danger': selected(i)}"
                 style="width: 240px;height: 360px; cursor:pointer">
              <img
                  :src="'/b4bimg/' + toSnakeCase(getCard(i).name) + '.png'"
                  class="card-img-top" alt="...">
              <div class="card-img-overlay">
                <h3 class="card-title text-white">{{ getCard(i).name }}</h3>
                <h6 class="card-subtitle mb-2 text-muted">{{ getCard(i).type }} | {{ getCard(i).affinity }}</h6>
                <p class="text-white" v-html="getCard(i).description" style="position: absolute;left:0.5rem;bottom: 0"></p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.bg-b4b-brawn {
  background-color: rgb(121 189 78);
}
.bg-b4b-discipline {
  background-color: rgb(194 56 44);
}
.bg-b4b-fortune {
  background-color: rgb(202 148 60);
}
.bg-b4b-reflex {
  background-color: rgb(98 125 179);
}
</style>

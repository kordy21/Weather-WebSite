<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input type="text" v-model="searchQuery" @input="getSearchReasults" placeholder="Search for a city or state" class="py-2
      px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]">

      <ul class="absolute bg-weather-secondary text-white w-full shadow-md  px-1 top-[66px] ">
        <p v-if="searchError">
          Sorry, something went wrong, please try again.
        </p>
        <p v-if="serverError && mapboxSearchResults.length === '0'">
          No results match query, try a different term.
        </p>
        <template v-else>
          <li class="py-2 cursor-pointer" 
          v-for="searchResult in mapboxSearchResults" 
          :key="searchResult.id" 
          @click="previewCity(searchResult)" >
            {{ searchResult.place_name }}
          </li>
        </template>
       
      </ul>
    </div>
  </main>
</template>

<script setup>
import { ref } from "vue";
import axios from 'axios';
import { useRouter } from "vue-router";

  const router = useRouter();
  const previewCity= (searchResult) =>{
    // console.log(searchResult);
    const [city , state] =searchResult.place_name.split(",");
    // console.log(city , state);
    router.push({
      name:"cityView",
      params: {state: state.replaceAll(" ","") , city:city},
      query:{
        lat: searchResult.geometry.coordinates[1], //latitude
        lng: searchResult.geometry.coordinates[0], // longitude
        preview:true
      }
    })

  }

  const mapboxAPIKey ="pk.eyJ1IjoibW9oYW1lZDE5OTkiLCJhIjoiY2xwMW1ucGtkMGcwcjJrcW9qbzB5dXQwMyJ9.I2RMdhZXBmb6dimgyUcqxw"
  const searchQuery = ref("");
  const queryTimeout = ref(null);
  const mapboxSearchResults =ref(null);
  const searchError =ref(null);

  const getSearchReasults = ()=>{
    clearTimeout(queryTimeout.value);
    queryTimeout.value = setTimeout( async ()=>{
      if(searchQuery.value !==""){
        try{
          const result = await axios.get(`https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`);
          mapboxSearchResults.value =result.data.features;
          // console.log(mapboxSearchResults.value)
        }
        catch{
          searchError.value=true;
        }
        return; 
      }
      mapboxSearchResults.value=null;
    },300)
  }
  
</script>


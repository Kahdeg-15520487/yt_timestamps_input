<template>

  <ul v-if="!loading && data && data.length">
    <li v-for="post in data" :key="post.id">
      <p>
        <strong>{{ post.videoId }}</strong>
      </p>
      <p>{{ post.date }}</p>
      <img v-bind:src="imgBaseUrl+post.videoId+'/0.jpg'" />
    </li>
  </ul>

  <p v-if="loading">Still loading..</p>
  <p v-if="error"></p>
</template>


<script>
import { ref, onMounted } from "vue";
export default {
  name: 'Posts',
  props: {
  },
  data(){
    return {
      imgBaseUrl: 'https://img.youtube.com/vi/'
    }
  },
  setup() {
    const data = ref(null);
    const loading = ref(true);
    const error = ref(null);

    function fetchData() {
      loading.value = true;
      // I prefer to use fetch
      // you can use use axios as an alternative
      return fetch('http://kahdeg.ddns.net/api/video', {
        method: 'get',
        headers: {
          'content-type': 'application/json'
        }
      })
        .then(res => {
          // a non-200 response code
          if (!res.ok) {
            // create error instance with HTTP status text
            const error = new Error(res.statusText);
            error.json = res.json();
            throw error;
          }

          return res.json();
        })
        .then(json => {
          // set the response data
          data.value = json;
          let lala = json.map((cv)=>{
            return {
              videoId : cv.videoId,
              date : (new Date(cv.startTime)).toLocaleString()
            };
          })
          data.value = lala;
        })
        .catch(err => {
          error.value = err;
          // In case a custom JSON error response was provided
          if (err.json) {
            return err.json.then(json => {
              // set the JSON response message
              error.value.message = json.message;
            });
          }
        })
        .then(() => {
          loading.value = false;
        });
    }

    onMounted(() => {
      fetchData();
    });

    return {
      data,
      loading,
      error
    };
  }
}
</script>

<style scoped>
p {
  text-align: left;
}
</style>
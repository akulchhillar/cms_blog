<template>
 
  <div class="mx-auto container flex flex-col justify-center items-center my-4">
    <Header />
    <Skeleton class="my-4" v-show="title==null"/>
    
  <div class="font-bold text-2xl">{{ title }}</div>
 
  <div class="prose prose-lg justify-center self-center my-4 p-4 prose-img:rounded-3xl" v-html="content"></div>
   <Footer/>
  </div>      

</template>

<script setup>
import { onMounted,ref } from 'vue';
import { useRoute } from 'vue-router'
import Skeleton from '@/components/Skeleton.vue'
import Header from '@/components/Header.vue'
import Footer from '@/components/Footer.vue'
import { annotate } from 'rough-notation';  

const route = useRoute()
const content = ref(null)
const title = ref(null)
const tags = ref([])

async function get_blog_content() {

const query = `
query Publication {
  publication(host: "akulchhillar.hashnode.dev") {
    isTeam
    title
    post(slug: "${route.params.slug}") {
      title
      readTimeInMinutes
      content {
        html,
        markdown
      }

    }
  }
}
`


  const data = await fetch('https://gql.hashnode.com',{
    method:'POST',
    headers: { 'Content-Type': 'application/json'},
    body: JSON.stringify({ query })

  })
  
  return data.json()
}

onMounted(()=>{
  
  get_blog_content().then((data)=>{
    
    if(data['data']['publication']['post']!=null){
    content.value = data['data']['publication']['post']['content']['html']
    title.value = data['data']['publication']['post']['title']
   

    

  }
  }).then(()=>{
let links = document.getElementsByTagName('a');

for (let i = 0; i < links.length; i++) {
  if(links[i].textContent != "The Quest of Akul"){
    const annotation = annotate(links[i], { type: 'highlight' ,color: '#FFF176'})
  annotation.show()
  
  }
  
}
  })




})


</script>
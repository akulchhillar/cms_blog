<template>
 
  <div class="mx-auto container flex flex-col justify-center items-center gap-4 my-4">
    <Header/>
    <Skeleton class="my-4" v-show="title==null"/>
  <div class="font-bold my-4">{{ title }}</div>
 
  <div class="prose prose-a:bg-[#66FCF1] prose-a:font-normal prose-a:underline
   prose-lg justify-center self-center my-4 prose-a:underline-offset-8 p-4" v-html="content"></div>
   <Footer/>
  </div>

</template>

<script setup>
import { onMounted,ref } from 'vue';
import { useRoute } from 'vue-router'
import Skeleton from '@/components/Skeleton.vue'
import Header from '@/components/Header.vue'
import Footer from '@/components/Footer.vue'

const route = useRoute()
const content = ref(null)
const title = ref(null)
const tags = ref([])
const query = `
query Publication {
  publication(host: "blog.akulchhillar.com") {
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
async function get_blog_content() {

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
  })



})


</script>
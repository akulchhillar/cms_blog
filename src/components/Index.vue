<template>
  
<div class="mx-auto container flex flex-col my-4 gap-10 justify-center items-center ">
<Header/>

  <div class="flex flex-col gap-6 lg:w-1/2 items-center px-2">
    <div class="text-xl font-semibold self-start"> My Thoughts</div>
    <div id="blogs" class=" flex flex-col gap-4 w-full">
    <div v-for="post in content" :key="post.node.slug" class="">
    
        <router-link :to="'/blog/' + post.node.slug" class="bg-[#66FCF1] 
         underline underline-offset-8 hover:bg-[#45C8C7] text-balance self-start">
            {{ post.node.title }}</router-link>

      
    </div>
  </div>

</div>

<button :disabled="has_next_page==false" class="disabled:bg-[#FAFAFA] disabled:border-0 disabled:hover:bg-[#FAFAFA]
     border-2 p-2 w-prose
    border-[#66FCF1] lg:hover:bg-[#66FCF1] md:active:bg-[#66FCF1] sm:active:bg-[#66FCF1] "
    :onclick="get_previous_posts">
    
        {{ btn_text }}
      </button>
    
      <Footer/>
</div>



</template>

<script setup>

import { onMounted,ref, watch } from 'vue';
import Footer from '@/components/Footer.vue'
import Header from '@/components/Header.vue'

let content = ref([])
let last_after = ref('')
let has_next_page = ref(null)
let btn_text = ref('Get Previous Posts')



async function get_posts(init=false) {

  let query = `query Publication {
  publication(host: "akulchhillar.hashnode.dev") {
    posts(
      first: 5
    ) {
      edges {
        node {
          title
          slug
          readTimeInMinutes
        }
        
      }
         pageInfo {
                endCursor
                hasNextPage
        				
            }
    }
  }
}`

if (init){
  query = `query Publication {
  publication(host: "akulchhillar.hashnode.dev") {
    posts(
      first: 5
      after: "${last_after.value}"
    ) {
      edges {
        node {
          title
          slug
          readTimeInMinutes
        }
        
      }
         pageInfo {
                endCursor
                hasNextPage
        				
            }
    }
  }
}`
}



const data = await fetch('https://gql.hashnode.com',{
  method:'POST',
  headers: { 'Content-Type': 'application/json'},
  body: JSON.stringify({ query })

})

return data.json()
}

function get_previous_posts(){

  if (has_next_page.value){
    get_posts(true).then((data)=>{
    content.value = content.value.concat(data['data']['publication']['posts']['edges'])
    last_after.value = data['data']['publication']['posts']['pageInfo']['endCursor']
    has_next_page.value = data['data']['publication']['posts']['pageInfo']['hasNextPage']
  })
  }
 

}

watch(has_next_page,async (val)=>{
  if (val==false) {
btn_text.value = "That's all folks!"
  }
});

onMounted(()=>{
    get_posts().then((data)=>{
   
    content.value = content.value.concat(data['data']['publication']['posts']['edges'])
    
   
    last_after.value = data['data']['publication']['posts']['pageInfo']['endCursor']
    has_next_page.value = data['data']['publication']['posts']['pageInfo']['hasNextPage']
    
    
  })



})

</script>

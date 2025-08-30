<template>
  
<div class="mx-auto container flex flex-col my-4 gap-10 justify-center items-center ">
<Header/>

  <div class="flex flex-col gap-6 lg:w-1/2 items-center px-2">
    <div class="text-xl font-semibold self-start"> My Thoughts</div>
    <div id="blogs" class=" flex flex-col gap-4 w-full">
    <div v-for="post in content" :key="post.node.slug" class="">
    
        <router-link :to="'/blog/' + post.node.slug" class="blog-link text-balance self-start overflow-hidden">
            {{ post.node.title }}</router-link>

      
    </div>
  </div>

</div>

<button :disabled="has_next_page==false" class="disabled:bg-[#FAFAFA] disabled:border-0 disabled:hover:bg-[#FAFAFA]
      p-2 w-prose
     lg:hover:bg-[#FFF176] md:active:bg-[#FFF176] sm:active:bg-[#FFF176] "
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
import { annotate } from 'rough-notation';  

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
  }).then(()=>{
let links = document.getElementsByClassName('blog-link');

for (let i = 0; i < links.length; i++) {
  if((links[i].textContent != "The Quest of Akul")&&(links[i].classList.contains('highlighted')==false)){
   
    const annotation = annotate(links[i], { type: 'highlight' ,color: '#FFF176'})
  annotation.show()
  
  }
  
}
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
    
    
  }).then(()=>{
let links = document.getElementsByClassName('blog-link');

for (let i = 0; i < links.length; i++) {
 
    const annotation = annotate(links[i], { type: 'highlight' ,color: '#FFF176'})
    links[i].classList.add('highlighted')
  annotation.show()
  
  
  
}

let footerlinks = document.getElementsByTagName('a');

for (let i = 0; i < footerlinks.length; i++) {
  if(footerlinks[i].textContent != "The Quest of Akul"){
   const annotation = annotate(footerlinks[i], { type: 'highlight' ,color: '#FFF176'})
   
  annotation.show()
  }
 
  
  
  
}
  })



})

</script>

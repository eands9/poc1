<template>
<div>Test</div>
</template>

<script>
export default {
    data(){
        return{
            students: []
        }
    },
    mounted(){
        this.getUpdates()
    },
    methods:{
        async getUpdates(){
          const endpoint='/data-api/graphql/Update'
          const response = await fetch(endpoint)
          const data = await response.json()
          console.log(data)
        }
    }
}
</script>
<template>
<div>
    <pageheader>
        <!--
        <b-input v-model="query" placeholder="Search ..."/>
        -->
    </pageheader>
    <sidemenu active="/pubs"></sidemenu>
    <div class="page-content">
        <b-container>
            <div v-if="!pubs" style="margin: 40px;"><h3>Loading ..</h3></div>
            <div v-else class="margin20">
                <div v-for="pub in pubs" :key="pub._id" class="pub">
                    <pubcard :pub="pub"/>
                </div>
            </div>
        </b-container>
    </div>
</div>
</template>

<script>
import Vue from 'vue'
import pageheader from '@/components/pageheader'
import sidemenu from '@/components/sidemenu'
import pubcard from '@/components/pubcard'

export default {
    components: { pageheader, sidemenu, pubcard },
    data () {
        return {
            pubs: [],
            config: Vue.config,
            //query: "",
        }
    },

    created: function() {
        this.$http.get('pub', {params: {
            find: JSON.stringify({ 
                removed: false 
            }),
            populate: 'project',
            deref_contacts: true,
        }})
        .then(res=>{
            this.pubs = res.body.pubs;
        }, res=>{
            console.error(res);
        });
    },

    methods: {
    }
}
</script>

<style scoped>
.pub {
margin-bottom: 10px;
}
</style>

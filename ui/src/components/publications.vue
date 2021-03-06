<template>
<div v-if="pubs">
    <div class="page-header with-menu header">
        <div style="margin-top: 2px; margin-left: 10px;">
            <b>{{pubs.length}}</b> Publications
        </div>
    </div>
    <div class="page-content with-menu content">
        <div v-if="publishing" style="background-color: white; padding: 20px;">
            <h3 style="opacity: 0.7;">New Publication</h3>
            <publisher :project="project" @close="publishing = false" @submit="publish"/>
        </div>
        <div v-else-if="pub_editing" style="background-color: white; padding: 20px;">
            <h3 style="opacity: 0.7;">{{pub_editing.doi||pub_editing._id+' (no doi)'}}</h3>
            <b-tabs class="brainlife-tab">
                <br>
                <b-tab title="Details">
                    <pubform :pub="pub_editing" @submit="save_pub" @cancel="cancel_pub"/>
                </b-tab>
                <b-tab title="Datasets">
                    <b-alert show variant="warning">Only the publication detail can be edited at this time. To update published datasets, please contact the administrator.</b-alert>
                     
                </b-tab>
            </b-tabs>
        </div>
        <div v-else>
            <!--list view-->
            <div v-for="pub in pubs" :key="pub._id" :class="{'pub-removed': pub.removed, 'pub-editable': (ismember()||isadmin())}" class="pub">
                <doibadge :doi="pub.doi" style="float: right;"/>
                <div class="pub-action">
                    <div class="button" @click="edit(pub)" title="Edit publication metadata"> <icon name="edit"/> </div>
                    <div class="button" @click="open_pub(pub)" title="See in published page"> <icon name="eye"/> </div>
                </div>
                <b-badge v-if="pub.removed" variant="danger">Removed</b-badge>
                <h5 style="margin-top: 10px;">
                    {{pub.name}}
                </h5>
                <p style="opacity: 0.7; margin-bottom: 5px;">
                    {{pub.desc}}
                </p>
                <p style="line-height: 180%; margin-bottom: 5px;" v-if="pub.tags.length > 0">
                    <small><tags :tags="pub.tags"/></small>
                </p>
                <!--<span style="float: right; opacity: 0.7;"><b>{{new Date(pub.publish_date||pub.create_date).toLocaleDateString()}}</b></span>-->
            </div>
            <div class="margin20 text-muted" v-if="!pubs || pubs.length == 0">
                <p>No publication registered for this project.</p>
                <p>To learn about how to submit publications, please refer to our <a href="https://brain-life.github.io/docs/user/publication/" target="doc">Documentation</a>.</p>
            </div>

            <!--space to make sure add button won't overwrap the pub list-->
            <p style="padding-top: 100px;">&nbsp;</p>

            <b-button v-if="isadmin() || ismember()" @click="start_publish" 
                class="button-fixed" 
                title="Create new publication">
                <icon name="plus" scale="2"/>
            </b-button>
        </div><!--pubs-->
    </div>

</div>
</template>

<script>
import Vue from 'vue'

import pubform from '@/components/pubform'
import publisher from '@/components/publisher'
import tags from '@/components/tags'
import doibadge from '@/components/doibadge'

import ReconnectingWebSocket from 'reconnectingwebsocket'

var debounce = null;

export default {
    props: [ 'project' ], 
    components: { 
        pubform, publisher, tags,
        doibadge,
    },
    data () {
        return {
            pubs: null, 

            publishing: false,
            pub_editing: null,

            config: Vue.config,
        }
    },

    mounted: function() {
        console.log("publications mounted", this.$route.params);
        this.load();
    },

    watch: {
        project: function() {
            console.log("project changed.. need to reload");
            this.load();
        },

        '$route': function() {
            var subid = this.$route.params.subid;
            if(!subid) this.pub_editing = null;
        },
    },

    methods: {
        load: function() {
            console.log("load all publication");
            this.$http.get('pub', {params: {
                find: JSON.stringify({
                    project: this.project._id,
                }),
                populate: 'project', //needed by pubcard
            }})
            .then(res=>{
                this.pubs = res.body.pubs; 

                if(this.$route.params.subid) {
                    this.pub_editing = this.pubs.find(pub=>{return pub._id == this.$route.params.subid});
                }
            });
        },

        isadmin() {
            if(!this.project) return false;
            if(~this.project.admins.indexOf(Vue.config.user.sub)) return true;
            return false;
        },

        ismember() {
            if(!this.project) return false;
            if(~this.project.members.indexOf(Vue.config.user.sub)) return true;
            return false;
        },

        notify_error: function(err) {
            console.error(err);
            this.$notify({type: 'error', text: err.body.message});
        },

        start_publish: function() {
            this.publishing = true;
        },

        save_pub: function(pub) {
            this.$http.put('pub/'+pub._id, pub)
            .then(res=>{
                this.$notify("Successfully updated!");
                for(var k in res.body) {
                    this.pub_editing[k] = res.body[k];
                }
                this.$router.push('/project/'+this.project._id+'/pub');
            }).catch(res=>{
                this.$notify({type: 'error', text: res.body});
            });
        },

        open_pub: function(pub) {
            document.location = "/pub/"+pub._id;
        },

        edit: function(pub) {
            if(this.ismember() || this.isadmin()) {
                this.$router.push("/project/"+this.project._id+"/pub/"+pub._id);
                this.pub_editing = pub;
            } 
        },

        cancel_pub: function() {
            console.log("cancel");
            this.$router.push("/project/"+this.project._id+"/pub/");
        },

        publish: function(pub) {
            this.publishing = false;
            pub.project = this.project; //pubcard needs project populated
            this.pubs.push(pub);
        }
    },
}

</script>

<style scoped>
.header {
top: 100px;
padding: 6px 10px;
color: #999;
background-color: #f9f9f9;
z-index: 1; /*needed to make sort order dropdown box to show up on top of page-content*/
height: 40px;
}
.content {
top: 100px;
margin-top: 40px;
padding-top: 10px;
}

.header, 
.content {
min-width: 500px;
}

.pub:first-child {
margin-top: 2px;
}
.pub {
padding: 5px 15px;
margin: 0px 20px;
background-color: white;
box-shadow: 1px 1px 3px rgba(0,0,0,0.3);
font-size: 88%;
}
.pub.pub-editable {
cursor: pointer;
}
.pub-action {
margin-right: 10px;
opacity: 0;
transition: 0.3s opacity;
float: right;
}
.pub:hover .pub-action {
opacity: 1;
}
</style>


<template>
<div>
    <pageheader/>
    <sidemenu active="/projects"></sidemenu>
    <div class="ui pusher">
        <div class="page-content">
        <div class="margin20">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item :to="{ path: '/projects' }">Projects</el-breadcrumb-item>
                <el-breadcrumb-item v-if="project._id" :to="{ path: '/project/'+project._id }">{{project._id}}</el-breadcrumb-item>
                <el-breadcrumb-item v-if="project._id">Edit</el-breadcrumb-item>
                <el-breadcrumb-item v-if="!project._id">New Project</el-breadcrumb-item>
            </el-breadcrumb>
            <br>

            <h1 v-if="$route.params.id == '_'">New Project</h1>
            <h1 v-else><icon name="edit" scale="2"/> Edit {{project.name}}</h1>

            <el-card>
                <el-form ref="form" label-width="120px">
                    <el-form-item label="Name">
                        <el-input type="text" v-model="project.name" placeholder="Project Name"></el-input>
                    </el-form-item>
                    <el-form-item label="Description">
                        <el-input type="textarea" v-model="project.desc" placeholder="Enter description for this project."></el-input>
                    </el-form-item>
                    <el-form-item label="Access">
                        <el-select v-model="project.access">
                            <el-option label="Private" value="private"></el-option>
                            <el-option label="Public" value="public"></el-option>
                        </el-select>
                        <p class="text-muted">Decide if non project member can access datasets inside this project</p>
                    </el-form-item>
                    <el-form-item label="Administrators">
                        <contactlist v-model="project.admins"></contactlist>
                        <p class="text-muted">Users who can update the project members</p>
                    </el-form-item>
                    <el-form-item label="Members">
                        <contactlist v-model="project.members"></contactlist>
                        <p class="text-muted">Users who can update the project members</p>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" icon="check" @click="submit()">Submit</el-button>
                    </el-form-item>
                </el-form>
            </el-card>
            
            <br>
            <el-card v-if="config.debug">
                <div slot="header">Debug</div>
                <h3>project</h3>
                <pre v-highlightjs="JSON.stringify(project, null, 4)"><code class="json hljs"></code></pre>
            </el-card>

        </div><!--margin20-->
        </div><!--page-content-->
    </div><!--page-->
</div>
</template>

<script>
import Vue from 'vue'

import sidemenu from '@/components/sidemenu'
import pageheader from '@/components/pageheader'
import contactlist from '@/components/contactlist'

export default {
    components: { sidemenu, contactlist, pageheader },
    data () {
        return {
            project: {
                _id: null,
                name: "",
                desc: "",
                access: "private",
                admins: [Vue.config.user.sub],
                members: [Vue.config.user.sub],
            },

            config: Vue.config,
        }
    },
    mounted: function() {
        /*
		$(this.$el).find('.repotype .item').tab({
            onVisible: (v)=> {
                this.repotype = v;
            }
        });
        */
        if(this.$route.params.id !== '_') {
            //load project to edit
            this.$http.get('project', {params: {
                find: JSON.stringify({_id: this.$route.params.id})
            }})
            .then(res=>{
                this.project = res.body.projects[0];
            });
        } else {
            //??
        } 
    },
    computed: {
    },
    methods: {
        changeadmins: function(admins) {
            //this.project.admins = admins;
        },

        add: function(it) {
            it.push({
                id: "",
                datatype: null,
                datatype_tags: [],
            });
        },

        submit: function() {
            if(this.project._id) {
                //update
                this.$http.put('project/'+this.project._id, this.project).then(res=>{
                    this.$router.push('/projects');
                }, res=>{
                    console.error(res);
                });
            } else {
                //create
                this.$http.post('project', this.project).then(res=>{
                    this.$router.push('/projects');
                }, res=>{
                    console.error(res);
                });
            }
        }
    },
}
</script>

<style scoped>
</style>



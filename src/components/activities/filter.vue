<template>
    <v-container class="ma-0 pa-0 pb-8">
        <v-row class="pt-10 px-8 pb-12">
            <strong>Filtros</strong>
            <v-spacer></v-spacer>
            <v-btn @click="removeFilter()" small class="elevation-0" style="border:solid 1px #dddddd;">
                <v-icon small class="mr-1">mdi-filter-off</v-icon>Limpiar
            </v-btn>

            <v-autocomplete @keydown.enter="filter()" v-model="activity.company_id" :items="companyLists" :loading="isLoadingCompany" :search-input.sync="searchCompanies" hide-no-data item-value="id" item-text="name" :label="liga!='https://backendbdb.unocrm.mx/'?'Empresa(s)':'Cliente'" placeholder="Escribe para buscar" attach chips multiple>
                <template v-slot:item="{item, attrs, on}">
                    <v-list-item v-on="on" v-bind="attrs">
                        <v-list-item-content>
                            <v-list-item-title v-if="item.name!=null">
                                <span v-if="item.macro!=null">{{item.macro}}</span>{{item.name}}
                                <div v-if="item.razon_social!=null">
                                    <span style="font-size:12px;">{{item.razon_social}}</span>
                                </div>
                            </v-list-item-title>
                            <v-list-item-title v-else-if="item.razon_social!=null">
                                {{item.razon_social}}
                            </v-list-item-title>
                        </v-list-item-content>
                    </v-list-item>
                </template> 
            </v-autocomplete>

            <v-autocomplete @keydown.enter="filter()" v-model="activity.contact_id" :items="contactList" :loading="isLoadingContact" :search-input.sync="searchContacts" hide-no-data item-value="id" item-text="name" label="Contacto(s)" v-if="liga!='https://backendbdb.unocrm.mx/'" placeholder="Escribe para buscar" attach chips multiple>
                <template v-slot:item="{item, attrs, on}">
                    <v-list-item v-on="on" v-bind="attrs">
                        <v-list-item-content>
                            <v-list-item-title>
                                {{item.name}} <span v-if="item.last!=undefined">{{item.last}}</span>
                                <div>
                                    <span style="font-size:12px;">{{item.company}}</span>
                                </div>
                            </v-list-item-title>
                        </v-list-item-content>
                    </v-list-item>
                </template> 
            </v-autocomplete>

            <v-autocomplete @keydown.enter="filter()" v-model="activity.sales_man" :items="userList" hide-no-data item-value="id" item-text="name" label="Vendedor(es)" placeholder="Escribe para buscar" attach chips multiple></v-autocomplete>
            <v-autocomplete @keydown.enter="filter()" v-model="activity.user_id" :items="userList" hide-no-data item-value="id" item-text="name" label="Usuario(s)" placeholder="Escribe para buscar" attach chips multiple></v-autocomplete>
            <v-autocomplete @keydown.enter="filter()" v-model="activity.created_by_user_id" :items="userList" hide-no-data item-value="id" item-text="name" label="Creador(es)" placeholder="Escribe para buscar" attach chips multiple></v-autocomplete>
            <v-autocomplete @keydown.enter="filter()" v-model="activity.last_updated_by_user_id" :items="userList" hide-no-data item-value="id" item-text="name" label="Editor(es)" placeholder="Escribe para buscar" attach chips multiple></v-autocomplete>

            <v-menu offset-y :close-on-content-click="closeDatePicker(activity.date)">
                <template v-slot:activator="{ on, attrs }">
                    <v-text-field clearable v-model="activity.date" label="Fecha" prepend-icon="mdi-calendar" v-bind="attrs" readonly v-on="on"></v-text-field>
                </template>
                <v-date-picker v-model="activity.date" range></v-date-picker>
            </v-menu>

            <v-menu offset-y :close-on-content-click="closeDatePicker(activity.created_at)">
                <template v-slot:activator="{ on, attrs }">
                    <v-text-field clearable v-model="activity.created_at" label="Fecha Creación" prepend-icon="mdi-calendar" v-bind="attrs" readonly v-on="on"></v-text-field>
                </template>
                <v-date-picker v-model="activity.created_at" range></v-date-picker>
            </v-menu>

            <v-menu offset-y :close-on-content-click="closeDatePicker(activity.updated_at)">
                <template v-slot:activator="{ on, attrs }">
                    <v-text-field clearable v-model="activity.updated_at" label="Fecha Edición" prepend-icon="mdi-calendar" v-bind="attrs" readonly v-on="on"></v-text-field>
                </template>
                <v-date-picker v-model="activity.updated_at" range></v-date-picker>
            </v-menu>

            <v-select @keydown.enter="filter()" label="Realizada" v-model="activity.completed" :items="statuses"></v-select>

            <v-text-field @keydown.enter="filter()" label="Descripción" v-model="activity.description"></v-text-field>

        </v-row>
        <v-card style="position:absolute; bottom:0; width:100%;" tile color="primary">
            <v-list-item link @click="filter()" dark>
                <v-list-item-content>
                    <strong>Filtrar (0)</strong>
                </v-list-item-content>
            </v-list-item>    
        </v-card>
    </v-container>
</template>

<script>
import axios from "axios";
export default {
    data: () => ({
        statuses:[{text:'Realizado', value:1}, {text:'No Realizado', value:null}],//{text:'Pendiente', value:'Pending'}, 
        activity:{
            //multiples axios
            company_id:[],
            contact_id:[],
            //multiples store
            user_id:[],
            created_by_user_id:'',
            last_updated_by_user_id:'',
            sales_man:[],
            //abierto
            description:'',
            //boolean
            completed:'',
            //date
            date:[],
            created_at:[],
            updated_at:[],
        },
        entries:{
            companies: [],
            contacts:[]
        },
        isLoadingCompany: false,
        isLoadingContact: false,
        searchCompanies: null,
        searchContacts: null,
    }),
    watch: {
        searchCompanies(val){
            //if (this.companyLists.length > 0) return
            if (this.isLoadingCompany) return
            this.isLoadingCompany = true
            axios.get(process.env.VUE_APP_BACKEND_ROUTE + 'api/v2/company_p?filter[name]='+val)
            .then(res => {
                this.entries.companies = res.data.data
            }).finally(() => (this.isLoadingCompany = false))
        },
        searchContacts(val){
            //if (this.contactList.length > 0) return
            if (this.isLoadingContact) return
            this.isLoadingContact = true

            var length = this.activity.company_id.length
            if(length>0){
                var filter = 'filter[company_id]='
                for(var i=0; i<length; i++){
                    filter = filter + this.activity.company_id[i]
                    if(length>1 && i<length-1){
                        filter = filter + ','
                    }else{
                        filter = filter + "&"
                    }
                }
            }else{
                var filter = ''
            }
            axios.get(process.env.VUE_APP_BACKEND_ROUTE + 'api/v2/contact_p?' + filter + "filter[name]=" + val)
            .then(res => {
                this.entries.contacts = res.data.data
            }).finally(() => (this.isLoadingContact = false))
        },
    },
    computed:{
        liga(){
            return process.env.VUE_APP_BACKEND_ROUTE
        },
        companyLists(){
            if(this.entries!=null){
                return this.entries.companies.map(id => {
                    return{
                        id:id.id,
                        macro:id.macro,
                        name:id.name,
                        razon_social:id.razon_social
                    }
                })
            }
        },
        contactList(){
            if(this.entries!=null){
                return this.entries.contacts.map(id => {
                    return{
                        id:id.id,
                        name:id.name,
                        last:id.last,
                        company:id.company.name,
                        job_position:id.job_position
                    }
                })
            }
        },
        userList(){
            return this.$store.state.user.users.map(id => {
                return{
                    id:id.id,
                    name:id.name + id.last,
                }
            })
        },
        currentUser:{
            get(){
                return this.$store.state.currentUser.user
            }
        },
    },
    created(){
        if(localStorage.getItem('filtersActivities')!=undefined){
            this.activity = JSON.parse(localStorage.getItem('filtersActivities'))
            this.entries = JSON.parse(localStorage.getItem('entriesFiltersActivities'))
        }
        if(this.currentUser.id!=1&&this.currentUser.id!=6){
          this.activity.user_id = [this.currentUser.id]
        }
        
    },
    mounted(){
        if(localStorage.getItem('filtersActivities')!=undefined){
            this.activity = JSON.parse(localStorage.getItem('filtersActivities'))
            this.entries = JSON.parse(localStorage.getItem('entriesFiltersActivities'))
        }
        if(this.currentUser.id!=1&&this.currentUser.id!=6){
          this.activity.user_id = [this.currentUser.id]
        }
    },
    methods:{
        closeDatePicker(dates){
            if(dates!=undefined && dates.length==2){
                return true
            }else{
                return false
            }
        },
        filter(){
            var count = 0
            var filter = ''
            //abierto
            if(this.activity.description!=''){
                count = count+1
                filter = filter + '&filter[description]='+this.activity.description
            }
            //multiples
            if(this.activity.company_id!=undefined && this.activity.company_id.length>0){
                count = count+1
                filter = filter + '&filter[company_id]='
                var length = this.activity.company_id.length
                for(var i=0; i<length; i++){
                    filter = filter + this.activity.company_id[i]
                    if(length>1 && i<length-1){
                        filter = filter + ','
                    }
                }
            }
            if(this.activity.contact_id!=undefined && this.activity.contact_id.length>0){
                count = count+1
                filter = filter + '&filter[contact_id]='
                var length = this.activity.contact_id.length
                for(var i=0; i<length; i++){
                    filter = filter + this.activity.contact_id[i]
                    if(length>1 && i<length-1){
                        filter = filter + ','
                    }
                }
            }
            if(this.activity.created_by_user_id!=undefined && this.activity.created_by_user_id.length>0){
                count = count+1
                filter = filter + '&filter[created_by_user_id]='
                var length = this.activity.created_by_user_id.length
                for(var i=0; i<length; i++){
                    filter = filter + this.activity.created_by_user_id[i]
                    if(length>1 && i<length-1){
                        filter = filter + ','
                    }
                }
            }
            if(this.activity.last_updated_by_user_id!=undefined && this.activity.last_updated_by_user_id.length>0){
                count = count+1
                filter = filter + '&filter[last_updated_by_user_id]='
                var length = this.activity.last_updated_by_user_id.length
                for(var i=0; i<length; i++){
                    filter = filter + this.activity.last_updated_by_user_id[i]
                    if(length>1 && i<length-1){
                        filter = filter + ','
                    }
                }
            }
            if(this.activity.user_id!=undefined && this.activity.user_id.length>0){
                count = count+1
                filter = filter + '&filter[user_id]='
                var length = this.activity.user_id.length
                for(var i=0; i<length; i++){
                    filter = filter + this.activity.user_id[i]
                    if(length>1 && i<length-1){
                        filter = filter + ','
                    }
                }
            }
            if(this.activity.sales_man!=undefined && this.activity.sales_man.length>0){
                count = count+1
                filter = filter + '&filter[company.user_id]='
                var length = this.activity.sales_man.length
                for(var i=0; i<length; i++){
                    filter = filter + this.activity.sales_man[i]
                    if(length>1 && i<length-1){
                        filter = filter + ','
                    }
                }
            }
            //select
            if(this.activity.completed!==''){
                count = count+1
                if(this.activity.completed==true){
                    filter = filter + '&filter[completed]='+this.activity.completed
                }else{
                    filter = filter + '&filter[completed]='+this.activity.completed+','+false
                }
            }
            //date
            if(this.activity.date!=undefined && this.activity.date.length==2){
                count = count+1
                filter = filter + '&filter[date_between]=' + this.activity.date[0] + ',' + this.activity.date[1]
            }
            if(this.activity.created_at!=undefined && this.activity.created_at.length==2){
                count = count+1
                filter = filter + '&filter[created_between]=' + this.activity.created_at[0] + ',' + this.activity.created_at[1]
            }
            if(this.activity.updated_at!=undefined && this.activity.updated_at.length==2){
                count = count+1
                filter = filter + '&filter[updated_between]=' + this.activity.updated_at[0] + ',' + this.activity.updated_at[1]
            }
            localStorage.setItem('selectorFiltersActivities', JSON.stringify(this.activity));
            localStorage.setItem('entriesFiltersActivities', JSON.stringify(this.entries));
            localStorage.setItem('filtersActivities', JSON.stringify(filter));
            localStorage.setItem('filtersActivitiesLength', count)

            this.$nextTick(() => {
                this.$emit("filtersActivities", false);
            })
        },
        removeFilter(){
            this.activity={
                //multiples axios
                company_id:'',
                contact_id:'',
                //multiples store
                user_id:'',
                created_by_user_id:'',
                last_updated_by_user_id:'',
                sales_man:'',
                //abierto
                description:'',
                //boolean
                completed:'',
                //date
                date:[],
                created_at:[],
                updated_at:[],
            },
            this.entries={
                companies: [],
                contacts:[]
            },
            localStorage.removeItem("selectorFiltersActivities")
            localStorage.removeItem("filtersActivities")
            localStorage.removeItem("entriesFiltersActivities")
            localStorage.removeItem("filtersActivitiesLength")
            this.$nextTick(() => {
                this.$emit("filtersActivities", false);
            })
        }
    }
}
</script>
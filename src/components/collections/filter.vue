<template>
    <v-container class="ma-0 pa-0">
        <v-row class="py-10 px-8">
            <strong>Filtros</strong>

            <div class="mt-4">
                Fecha 
                <v-menu v-model="menuFrom" :close-on-content-click="false" :nudge-right="40" transition="scale-transition" offset-y min-width="290px" >
                    <template v-slot:activator="{ on }">
                        <v-text-field @keydown.enter="filter()" v-model="quotation.dateFrom" label="Desde" clearable prepend-icon="mdi-calendar-today" readonly v-on="on"></v-text-field>
                    </template>
                    <v-date-picker color="primary" v-model="quotation.dateFrom" @input="menuFrom = false"></v-date-picker>
                </v-menu>
                <v-menu v-model="menuTo" :close-on-content-click="false" :nudge-right="40" transition="scale-transition" offset-y min-width="290px" >
                    <template v-slot:activator="{ on }">
                        <v-text-field @keydown.enter="filter()" v-model="quotation.dateTo" label="Hasta" clearable prepend-icon="mdi-calendar" readonly v-on="on"></v-text-field>
                    </template>
                    <v-date-picker color="primary" v-model="quotation.dateTo" @input="menuTo = false"></v-date-picker>
                </v-menu>
            </div>

            <v-select v-model="quotation.macro" clearable :items="macros" label="Macro"></v-select>

            <v-autocomplete v-if="company==undefined" @keydown.enter="filter()" v-model="quotation.company_id" :items="companyLists" item-text="name" item-value="id" attach chips label="Empresa" multiple>
                <template slot="no-data" class="pa-2">No existen empresas relacionadas.</template>  
            </v-autocomplete>

            <!--v-autocomplete @keydown.enter="filter()" item-text="name" v-model="quotation.contact_id" :items="contactLists" item-value="id" attach chips label="Contacto" multiple>
                <template slot="no-data" class="pa-2">No existen contactos relacionados.</template>  
                <template v-slot:item="{item, attrs, on}">
                    <v-list-item v-on="on" v-bind="attrs" #default="{ active }">
                        <v-list-item-action class="mr-2">
                            <v-checkbox :input-value="active"></v-checkbox>
                        </v-list-item-action>
                        <v-list-item-content>
                            <v-list-item-title>
                                {{item.name}} <v-chip v-if="quotation.company_id==null || quotation.company_id.length>1 || quotation.company_id.length==0" small class="ml-2"  style="font-size:8px;">{{companyName(item.company_id)}}</v-chip>
                            </v-list-item-title>
                        </v-list-item-content>
                    </v-list-item>
                </template> 
            </v-autocomplete-->

            <v-text-field @keydown.enter="filter()" v-model="quotation.invoice" label="Factura"></v-text-field>
            <v-text-field @keydown.enter="filter()" v-model="quotation.remission" label="Remisión"></v-text-field>
            <v-autocomplete @keydown.enter="filter()" v-model="quotation.user_id" :items="userLists" item-text="name" item-value="id" attach chips label="Vendedor" multiple>
            </v-autocomplete>

            <v-autocomplete @keydown.enter="filter()" v-model="quotation.method_id" :items="methodLists" item-text="method" item-value="id" attach chips label="Metodo de Pago" multiple>
            </v-autocomplete>


            <v-text-field @keydown.enter="filter()" v-model="quotation.note" label="Referencia"></v-text-field>

            <v-autocomplete @keydown.enter="filter()" v-model="quotation.sale_id" :items="salesLists" attach chips label="Id de Venta" multiple>
                <template slot="no-data" class="pa-2">No existen ventas relacionadas.</template>  
            </v-autocomplete>

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
  export default {
    props:{
        company:Object
    },
    data: () => ({
        macros:[{text:'si', value:1}, 
        {text:'no', value:0}],
        menuTo:false,
        menuFrom:false,
        quotation:{
            macro:'',
            company_id:null,
            //contact_id:'',
            note:'',
            user_id:'',
            sale_id:'',
            dateTo:'',
            dateFrom:'',
            invoice:'',
            remission:'',
            method_id:'',
        },
    }),
    computed: {
        methodLists(){
            return this.$store.state.payment_method.payment_methods;
        },
        companyLists(){
            return this.$store.state.company.companies.map(id=>{
                return{
                    id: id.id,
                    name: this.number(id.number) + id.name 
                }
            })
        },
        contactLists(){
            if(this.quotation.company_id!=null){
                if(this.quotation.company_id.length==0){
                    return this.$store.state.contact.contacts
                }
                else{
                    var cuatro = this.$store.state.contact.contacts.filter(contact=>contact.company_id == this.quotation.company_id[0]);
                    for(var i=1; i<this.quotation.company_id.length; i++){
                        cuatro=cuatro.concat(this.$store.state.contact.contacts.filter(contact=>contact.company_id == this.quotation.company_id[i]))   
                    }
                    return  cuatro
                }
            }
            else{
                return this.$store.state.contact.contacts
            }
        },
        userLists(){
            return this.$store.state.user.users;
        }, 
        salesLists(){
            var collections = this.$store.state.collection.collections
            var response = []
            for(var i=0; i<collections.length; i++){
                for(var j=0; j<collections[i].salesID.length; j++){
                    response = response.concat(collections[i].salesID[j].id)
                }
            }
            return response
        }
    },
    created () {
        
    },
    methods: {
        number(id){
            if(id!=null&&id!=''&&id!=undefined){
                return id + ' | '
            }else{
                return ''
            }
        },
        companyName(id){
            return this.$store.state.company.companies.filter(company=>company.id == id).map(company=>company.name)[0];
        },
        filter(){
            this.$emit("filtersCollection", this.quotation);
        }
    }
  }
</script>
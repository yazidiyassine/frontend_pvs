<template>
    <div>
        الشكايات
        <v-data-table
    :headers="headers"
    :items="plaintes"
    :loading="load_tab"
    hide-default-footer
    loading-text="إنتظر قليلا"
    no-data-text="لاتوجد شكايات"
  >
  <template v-slot:top>
  <v-dialog v-model="dialog" max-width="500px">
          <v-card :loading="load_vcard">
              <v-container>
                <v-form ref="form" v-model="valideform">
                    <v-text-field
                      :rules="nameRules"
                      v-model="userhasplaintes.descision"
                      label="القرار"
                    ></v-text-field>
                </v-form>
                    
              </v-container>

              <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closedialog">
                رجوع
              </v-btn>
              <v-btn color="blue darken-1" text :disabled="!valideform" @click="valider_edite_descision" >
                تغيير
              </v-btn>

            </v-card-actions>
          </v-card>
        </v-dialog>
  </template>
  <template v-slot:[`item.lien`]="{item}">
      <v-chip
      small
      @click="redirect(item.lien)"
      color="blue lighten-5"
      > تصفح
      <v-icon small>mdi-download</v-icon></v-chip>
    </template>

    <template v-slot:[`item.actions`]="{item}">
        <v-chip 
      small 
      @click="edit_item(item)"
      :disabled="item.traitID == 2 ? true : false">
      توقيع
      <v-icon small>mdi-pencil</v-icon>
      </v-chip>
    </template>

  </v-data-table>
    </div>
</template>

<script>
import axios from 'axios';
import baseURL from '@/api/baseURL';

export default{
    data(){
        return {
            headers: [
        { text: 'مرجع الشكاية', align: 'start', sortable: false, value: 'referencePlaints'},
        { text: 'تاريخ التسجيل', value: 'dateEnregPlaints', sortable: true },
        { text: 'تاريخ الاحالة', value: 'dateMission', sortable: true },
        { text: 'الملف', value: 'lien',sortable: false },
        { text: 'القرار', value: 'actions', sortable: false },
        
      ],
            plaintes:[],       // pours les donnees de tableau
            dialog:false, // affichage de dialog
            load_tab:false, // load tableau vide des plaintes
            userhasplaintes:{     // edited item ogenerer new pdf au backend
                descision:'',
                traitID:2,
                lien:'',
                idplainte:null,
            },
            valideform:true,
            nameRules: [(v) => !!v || "حقل ضروري"],
            load_url:false, // ici loader application
            load_vcard:false, // lorsque on veut ajouter la descision

        }
    },

    methods:{
        // fermer le dialog
         closedialog(){
            this.dialog = false ;
            this.userhasplaintes.lien ='';
          this.userhasplaintes.idplainte = -1;
          this.userhasplaintes.descision = '';
        },

        // navigate to document
        redirect(link) { 
          var names = link.split('/')
          var fileLink = document.createElement('a');
                fileLink.href =  baseURL.backendPDF+"/dossiers_plaintes/"+names[2];
                fileLink.target = "_blank"; 
                fileLink.click();
              
      },
        // changer la descision dans la table userhasplaintes(descision)
        edit_item(item){
          this.dialog= true;
          this.userhasplaintes.lien = item.lien;
          this.userhasplaintes.idplainte = item.id;
        },
      
        valider_edite_descision(){
          this.load_vcard = true;
                axios
            .post(baseURL.api + "/users/hasplaints/signer_plainte/"+this.userhasplaintes.idplainte,{
              userhasplaint:this.userhasplaintes
            },{
              headers: { Authorization: `Bearer ${baseURL.token}` },
            })
            .then(async(rep) => {
              if(rep.status == 200){
                await this.get_mes_plaintes();
                this.closedialog();
                this.load_vcard = false;
              }else{
                this.load_vcard = false;
              }
            }).catch(erreur=>{
              this.load_vcard = false;
            });
        },

        get_mes_plaintes(){
          this.load_tab = true;
        axios
      .get(baseURL.api + "/users/hasplaints/mesplaintes", {
        headers: { Authorization: `Bearer ${baseURL.token}` },
      })
      .then((rep) => {
        this.plaintes = rep.data;
        this.load_tab=false;
      }).catch(erreur=>{
        this.load_tab = false;
      });
        }
       
    },

    mounted(){ 
      this.get_mes_plaintes();// les plaintes de ce utilisateur
    }
}
</script>
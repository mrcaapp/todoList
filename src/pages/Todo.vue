<template>
  <q-page class="bg-grey-3 column">
    <div separator class="row q-pa-sm bg-info">
      <q-input 
        @keyup.enter="addTarefa(novaTarefa)" class="col" bg-color="white" filled v-model="novaTarefa" id="terefinha"  label="Adicionar tarefa" placeholder="Digite a nova tarefa" dense>
        <template v-slot:append>
          <q-btn 
          @click="addTarefa"
          round dense flat icon="add" />
        </template>
      </q-input>
    </div>
    <q-scroll-area :thumb-style="thumbStyle" :bar-style="barStyle" style="height: 515px; max-width: 100%;">
      <q-list class="bg-white" separator bordered>
        <q-item
          v-for="(tarefa, index) in tarefas"
          :key="index"
          @click="abriDialogTarefa(tarefa)"
          :class="{ 'feito bg-green-1' : tarefa.doc.status}"
          clickable 
          v-ripple>
          <q-item-section avatar>
            <q-checkbox @input="concluiTarefa(tarefa.doc)" v-model="tarefa.doc.status" class="no-poiner-events" color="primary" />
          </q-item-section>
          <q-item-section>
            <q-item-label>{{tarefa.doc.titulo}}</q-item-label>
          </q-item-section>
          <q-item-section side>
            <q-btn @click.stop="apagaTarefa(tarefa.doc)" flat dense round color="negative" icon="delete" />
          </q-item-section>
        </q-item>
    </q-list>
    </q-scroll-area>
    <div v-if="!tarefas.length" class="no-tasks absolute-center">
      <q-icon name="check" size="120px" color="primary" side></q-icon>
      <div class="text-h5 text-primary text-center">
        Nenhuma
        <br>
        tarefa
      </div>
    </div> 
    <!-- Dialog Subtarefa -->
    <q-dialog v-model="showAddSubtask">
      <q-card style="height: 64 vh; width: 100%;">
        <q-card-section class="row">
          <div class="text-h6">Adicionar Subtarefa</div>
          <q-space/>
          <q-btn v-close-popup round dense icon="close" />
        </q-card-section>
        <q-separator/>
        <q-card-section style="height: 52
        vh;" class="q-pt-none">
          <br>
          <q-input @keyup.enter="addSubtarefa" class="col" bg-color="white" filled v-model="novaSubtarefa" id="terefinha" placeholder="Digite a nova subtarefa" dense>
            <template v-slot:append>
              <q-btn @click="addSubtarefa" round dense flat icon="add" />
            </template>
          </q-input>
          <br>
          <q-list class="bg-white" separator bordered>
            <q-scroll-area :thumb-style="thumbStyle" :bar-style="barStyle" style="height: 32vh; max-width: 100vw;">
              <q-item v-for="(subTarefa, index) in dadosModal.subtarefa" :key="index" clickable v-ripple :class="{ 'feito bg-green-1' : subTarefa.status}">
                <q-item-section avatar>
                  <q-checkbox @input="concluiSub()" v-model="subTarefa.status" class="no-poiner-events" color="primary"/>
                </q-item-section> 
                <q-item-section>
                  <q-item-label>{{subTarefa.tituloSub}}</q-item-label>
                </q-item-section>    
                <q-item-section side>
                  <q-btn @click="abrirDialogAnexo(subTarefa)" dense round color="primary" icon="attachment">
                    <q-badge v-if="subTarefa.anexo.length" color="red" floating>{{subTarefa.anexo.length}}</q-badge>
                  </q-btn>
                </q-item-section>
                <q-item-section side>
                  <q-btn @click.stop="apagaSubtarefa()" flat round color="negative" icon="delete"/>
                </q-item-section>
                <q-separator/>
              </q-item>
            </q-scroll-area>  
          </q-list>
          <q-card-actions align="right">
            <q-btn flat label="OK" color="primary" v-close-popup />
          </q-card-actions>
          <div v-if="!dadosModal.subtarefa.length" class="no-tasks absolute-center">
            <q-icon name="check" size="120px" color="primary" side></q-icon>
            <div class="text-h5 text-primary text-center">
              Nenhuma
              <br>
              Subtarefa
            </div>
          </div> 
        </q-card-section>
      </q-card>
    </q-dialog> 
    <!-- Dialog anexo -->
    <q-dialog v-model="showAddAnexo">
      <q-card style="height: 64vh; width: 100%;">
        <q-card-section class="row">
          <div class="text-h6">Anexos</div>
          <q-space/>
          <q-btn v-close-popup round dense icon="close" />
        </q-card-section>
        <q-separator/>
        <q-card-section class="q-pt-none">
          <q-file v-model="file" label="selecionar" style="max-width: 100vw; max-height: 61px; overflow: hidden;" clearable>
            <template v-slot:after>
              <q-btn @click="filepicked(file)" round dense flat icon="send" />
            </template>
          </q-file>
          <br>
          <q-list class="bg-white" separator bordered>
            <q-scroll-area :thumb-style="thumbStyle" :bar-style="barStyle" style="height: 32vh; max-width: 100vw;">
              <q-item v-for="(anexoView, index) in dadosSub.anexo" :key="index" clickable v-ripple>
                <q-item-section>
                  <q-slide-item @left="onLeft($event, anexoView)" @right="onRight($event, anexoView)" right-color="red" left-color="primary">
                    <template v-slot:right>
                      <div class="row items-center">
                        <q-icon left name="clear" /> Apagar
                      </div>
                    </template>
                    <template v-slot:left>
                      <div class="row items-center">
                        <q-icon left name="download" /> Download
                      </div>
                    </template>
                    <q-item>
                      <q-item-section>
                        <q-btn disable>{{anexoView._attachments.nomeArq}}</q-btn>
                      </q-item-section>
                    </q-item>
                  </q-slide-item>
                </q-item-section>    
                <q-separator/>
              </q-item>
            </q-scroll-area>  
          </q-list>
          <q-card-actions align="right">
            <q-btn flat label="OK" color="primary" v-close-popup />
          </q-card-actions>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page> 
</template>

<script>
import PouchDB from 'pouchdb'
import { date } from 'quasar'
// 
export default {
  name: 'PageIndex',
  data(){
    return{
      file: null,
      oldPickedFile: null,
      db: new PouchDB('todo'),
      /////////////////////////////////////////////////////////
      dadosModal: {subtarefa:[]},
      dadosSub: {anexo:[]},
      novaTarefa: '',
      novaSubtarefa: [],
      showAddSubtask: false,
      showAddAnexo: false,
      tarefas:[],
      thumbStyle: {
        right: '4px',
        borderRadius: '5px',
        backgroundColor: '#027be3',
        width: '5px',
        opacity: 0.75
      },
      barStyle: {
        right: '2px',
        borderRadius: '9px',
        backgroundColor: '#027be3',
        width: '9px',
        opacity: 0.2
      }
    }
  },
  mounted(){
    this.showTodos()
  },  
  methods: {
    filepicked(file) {  
      try{
        if (file !== void 0) {
          this.oldPickedFile = file
          // debugger
        } 
        else {
          this.file = this.oldPickedFile
          // debugger
        }
        var reader = new FileReader();
        reader.onload = (event) => {
         var arq = event.target.result
         var anexo = {
          _attachments: {
            base64: arq,
            nomeArq: file.name,
            tipoArq: file.type
          }
        }
        this.dadosSub.anexo.push(anexo)
        // debugger
        this.db.put(this.dadosModal)
        console.log("deu bom")  
        }
        reader.readAsDataURL(file)
        
        // debugger
        
      }catch(err){
        debugger
        console.log("deu ruim")
        console.log(err)
      }
      this.showTodos();
    },
    concluiTarefa(doc){
      this.db.put(doc)
    },
    concluiSub(){
      this.db.put(this.dadosModal)
      console.log('deu bom')
    },
    addTarefa(novaTarefa){
      if(!this.novaTarefa.length){
          this.$q.dialog({
          title: 'Alerta',
          message: 'Por favor, digite uma Tarefa'
        })
      }
      else{
        var todo = {
          _id: new Date().toISOString(),
          titulo: novaTarefa,
          status: false,
          subtarefa: []
        }

      }this.db.put(todo)
      .then(function(result){
        console.log('Sucesso')
        console.log(todo.titulo)
      })
      .catch(function(err){
        console.log('Erro!');
        console.log(err);
      });
        this.novaTarefa=""
        this.showTodos()
    },
    showTodos() {
      this.db.allDocs({include_docs: true, descending: true})
      .then((doc) => {
        this.tarefas = doc.rows
      }).catch(function (err) {
        console.log(err)
      });
    },
    addSubtarefa(){
      if(!this.novaSubtarefa.length){
          this.$q.dialog({
          title: 'Alerta',
          message: 'Por favor, digite uma subtarefa',
        })
      }
      else{
        var todo = {
          tituloSub: this.novaSubtarefa,
          status: false,
          anexo: []
        }
        //_rev = this.dadosModal._rev
      }
      this.dadosModal.subtarefa.push(todo)
        try{
          this.db.put(this.dadosModal)
          console.log('deu bom')
        }catch(err){
          console.log('deu ruim')
          console.log(err)
        }
      this.novaSubtarefa=""
      this.showTodos()
    },
    apagaTarefa(dados){
      this.$q.dialog({
        title: 'Confirmar',
        message: 'Deseja mesmo apagar a tarefa?',
        cancel: true,
        persistent: true
      }).onOk(() => {
        this.db.remove(dados)
        this.showTodos()
        this.$q.notify('Tarefa apagada com sucesso')
      })
    },
    apagaSubtarefa(index){
      this.$q.dialog({
        title: 'Confirmar',
        message: 'Deseja mesmo apagar a Subtarefa?',
        cancel: true,
        persistent: true
      }).onOk(() => {
        this.dadosModal.subtarefa.splice(index, 1),
        this.db.put(this.dadosModal)
        this.$q.notify('Subtarefa apagada com sucesso')
      })
    },
    abriDialogTarefa (tarefa) {      
      this.showAddSubtask = true
      this.dadosModal = tarefa.doc
    },
    abrirDialogAnexo (subtarefa) {      
      this.showAddAnexo = true
      this.dadosSub = subtarefa
    },
   
    onLeft ({ reset }, anexo) {
      this.$q.notify('Iniciando download')
      this.finalize(reset)
      var content = anexo;
      const link = anexo._attachments.base64;
      debugger
      const a = document.createElement('a');
      a.style.display = 'none';
      a.href = link;
      a.download = content._attachments.nomeArq;
      document.body.appendChild(a);
      a.click();
      window.URL.revokeObjectURL(link)
    },

    onRight ({ reset }, index) {
      this.dadosSub.anexo.splice(index, 1),
      this.db.put(this.dadosModal)
      this.$q.notify('Arquivo deletado!')
      this.finalize(reset)
    },

    finalize (reset) {
      this.timer = setTimeout(() => {
        reset()
      }, 1000)
    }  
  },
}
</script>
<style lang="scss" scoped>
  .feito{
    .q-item__label{
      text-decoration: line-through;
      color: #bbb;
    }
  }
  .no-tasks{
    opacity: 0.5;
  }
</style>
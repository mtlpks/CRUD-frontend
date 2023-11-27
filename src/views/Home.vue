<template>
  <v-layout style="height: 100vh; width: 100vw;">

    <!--NAVBAR-->
    <v-app-bar color="primary" prominent>
      <v-app-bar-nav-icon variant="text" @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>CRUD</v-toolbar-title>
      <v-spacer></v-spacer>
    </v-app-bar>

    <!--SELECAO DE OPERACAO CRUD-->
    <v-navigation-drawer class="h-auto" v-model="drawer" location="left" temporary>
      <v-list v-model="selecaoOperacao">
        <v-list-item v-for="item in items" :key="item.value" @click="selectOperation(item.value)">
          <v-list-item-title>{{ item.title }}</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>


    <v-main class="h-auto justify-center" style="height: 900px;">
      <v-container class="h-auto d-flex justify-center" style="width: 80%;">
        
        <!--CREATE-->
        <v-row class="d-flex justify-center" v-if="selecaoOperacao === 'create'">
          <h1>Adicionar cliente - Create</h1>
          <v-row class="d-flex flex-column" style="min-width: 80%;">
            <v-text-field v-model="dataCliente.nome" label="Nome"></v-text-field>
            <v-text-field v-model="dataCliente.telefone" label="Telefone"></v-text-field>
            <v-text-field v-model="dataCliente.email" label="Email"></v-text-field>
            <v-text-field v-model="dataCliente.endereco" label="Endereço"></v-text-field>
            <v-btn style="width: 150px;" color="primary" @click="adicionaCliente">Adicionar</v-btn>
          </v-row>
        </v-row>

        <!--READ-->
        <v-row class="d-flex justify-center" v-else-if="selecaoOperacao === 'read'">
          <v-row class="d-flex flex-column mt-8">
            <h1>Retornar - Read</h1>
            <v-btn color="primary" @click="selecionaGetAll">Retornar todos</v-btn>
            <v-btn color="primary" @click="selecionaGetOne">Retornar um</v-btn>
          </v-row>
          <v-row v-if="selecaoGet === 'retornaTodos'">
            <h1>Retornar clientes</h1>
            <v-data-table :items="respostaApiGetAll"></v-data-table>
          </v-row>
          <v-row class="d-flex flex-column" v-else-if="selecaoGet === 'retornaUm'">
            <h1>Retornar cliente</h1>
            <v-text-field v-model="idRequest" type="number" label="ID"></v-text-field>
            <v-btn color="primary" @click="retornaCliente(idRequest)">Retornar por ID</v-btn>
            <v-data-table :items="respostaApiGetOne" mobile-break-point="1264"></v-data-table>
          </v-row>
        </v-row>

        <!--UPDATE-->
        <v-row class="d-flex flex-column align-center" v-else-if="selecaoOperacao === 'update'">
          <h1>Atualizar cliente - Update</h1>
          <v-row class="d-flex flex-column">
            <v-text-field v-model="dataAtualizaCliente.id" type="number" label="ID"></v-text-field>
            <v-text-field v-model="dataAtualizaCliente.nome" label="Nome"></v-text-field>
            <v-text-field v-model="dataAtualizaCliente.telefone" label="Telefone"></v-text-field>
            <v-text-field v-model="dataAtualizaCliente.email" label="Email"></v-text-field>
            <v-text-field v-model="dataAtualizaCliente.endereco" label="Endereço"></v-text-field>
            <v-btn color="primary" @click="atualizaCliente">Atualiza cliente</v-btn>
          </v-row>
        </v-row>

        <!--DELETE-->
        <v-row class="d-flex flex-column" v-else-if="selecaoOperacao === 'delete'">
          <h1>Deletar cliente - Delete</h1>
          <v-text-field v-model="idDelete" label="ID"></v-text-field>
          <v-btn color="primary" @click="deletaCliente(idDelete)">Deletar cliente</v-btn>
        </v-row>

        <!--PAGINA INICIAL-->
        <v-row v-else class="flex-column">
          <h1>Pagina Inicial</h1>
          <v-row>
            <v-btn color="primary" @click="selecaoOperacao = 'create'">Adicionar Cliente</v-btn>
            <v-btn color="primary" @click="selecaoOperacao = 'read'">Retorna Cliente</v-btn>
            <v-btn color="primary" @click="selecaoOperacao = 'update'">Atualiza Cliente</v-btn>
            <v-btn color="primary" @click="selecaoOperacao = 'delete'">Deleta Cliente</v-btn>
          </v-row>
        </v-row>
      </v-container>
    </v-main>
  </v-layout>
</template>


// Utilizando Options API
<script lang="ts">
import axios, { AxiosResponse, AxiosError } from 'axios';
axios.defaults.baseURL = 'http://localhost:5199/api/cliente/'
export default {
  data() {
    return {
      respostaApiGetAll: [] as any[],
      respostaApiGetOne: [] as any[],
      urlAPI: "http://localhost:5199/api/cliente/",
      dataCliente: {
        nome: '',
        telefone: '',
        email: '',
        endereco: ''
      },
      dataAtualizaCliente: {
        id: 0,
        nome: '',
        telefone: '',
        email: '',
        endereco: ''
      },
      idRequest: 0,
      idUpdate: 0,
      idDelete: 0,
      selecaoOperacao: '',
      selecaoGet: '',
      drawer: false,
      items: [
        {
          title: 'Pagina Inicial',
          value: 'inicial'
        },
        {
          title: 'Adicionar',
          value: 'create',
        },
        {
          title: 'Retornar',
          value: 'read',
        },
        {
          title: 'Atualizar',
          value: 'update',
        },
        {
          title: 'Deletar',
          value: 'delete'
        }
      ]
    };
  },
  methods: {
    selecionaGetAll() {
      this.selecaoGet = 'retornaTodos';
      this.retornaClientes();
    },
    selecionaGetOne() {
      this.selecaoGet = 'retornaUm';
    },
    selectOperation(value: any) {
      this.selecaoOperacao = value;
      this.drawer = false;
    },
    
    //metodo CREATE
    adicionaCliente() {
      axios.post(this.urlAPI, this.dataCliente)
        .then(response => {
          this.dataCliente = {
            nome: '',
            telefone: '',
            email: '',
            endereco: ''
          }
        })
        .catch(error => {
          console.log(error);
        })
    },

    //metodo READ (all)
    retornaClientes() {
      axios.get(this.urlAPI)
        .then(response => {
          this.respostaApiGetAll = response.data;
        })
        .catch(error => {
          console.log(error);
        })
    },

    //metodo READ (one)
    retornaCliente(idCliente: number) {
      axios.get(`${this.urlAPI}${idCliente}`)
        .then(response => {
          this.respostaApiGetOne = []
          this.respostaApiGetOne.push(response.data)
        })
        .catch(error => {
          console.log(error);
        })
    },

    //metodo UPDATE
    atualizaCliente() {
      const intId = Number(this.dataAtualizaCliente.id)
      axios.put(`${this.urlAPI}${intId}`, this.dataAtualizaCliente)
        .then(response => {
          this.dataAtualizaCliente = {
            id: 0,
            nome: '',
            telefone: '',
            email: '',
            endereco: ''
          }
        })
        .catch(error => {
          console.log(error)
        })
    },

    //metodo DELETE
    deletaCliente(idCliente: number) {
      axios.delete(`${this.urlAPI}${idCliente}`)
        .then(

      )
    },
  }
}
</script>

<style>
.v-text-field {
  min-width: 400px;
  max-height: 80px;
}

.v-row {
  gap: 10px;
  width: auto;
  flex: 0 1 auto !important;
  ;
}

.v-btn {
  min-width: 300px;
}

h1 {
  text-align: center;
  margin-top: 2rem;
  margin-bottom: 2rem;
}

@media only screen and (max-width: 1000px) {
  .v-text-field {
    min-width: 300px;
  }
}
</style>
<template>
  <div id="app">
    <h1>Lista de Pacientes</h1>
    <button class="add-button" @click="addPaciente">Adicionar Paciente</button>
    <table class="pacientes-table">
      <thead>
        <tr>
          <th>Nome</th>
          <th>Sexo</th>
          <th>Idade</th>
          <th>Resultado Ex1</th>
          <th>Resultado Ex2</th>
          <th>Ações</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(paciente, index) in pacientes" :key="paciente.id">
          <td>{{ paciente.nome }}</td>
          <td>{{ paciente.sexo }}</td>
          <td>{{ paciente.idade }}</td>
          <td>{{ paciente.valorResultadoEx1 }}</td>
          <td>{{ paciente.valorResultadoEx2 }}</td>
          <td>
            <button class="edit-button" @click="editPacientes(index)">Editar</button>
            <button class="delete-button" @click="deletePacientes(paciente.id)">Excluir</button>
            <button class="delete-button" @click="modalCalculo(index)">Calculo</button>
          </td>
        </tr>
      </tbody>
    </table>

    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <label for="nome">Nome:</label>
        <input type="text" v-model="model.paciente.nome" placeholder="Nome">
        <span class="error-message" v-if="!isValidNome">Nome é obrigatório</span><br><br>
        <label for="sexo">Sexo:</label>
        <input type="text" v-model="model.paciente.sexo" placeholder="Sexo">
        <span class="error-message" v-if="!isValidSexo">Sexo deve ser 'M' ou 'F'</span><br><br>
        <label for="idade">Idade:</label>
        <input type="text" v-model="model.paciente.idade" placeholder="Idade">
        <span class="error-message" v-if="!isValidIdade">Idade é obrigatória</span><br><br>
        <label for="ex1">Resultado Ex1:</label>
        <input type="text" v-model="model.paciente.valorResultadoEx1" disabled placeholder="Resultado Ex1">
        <label for="ex2">Resultado Ex2:</label>
        <input type="text" v-model="model.paciente.valorResultadoEx2" disabled placeholder="Resultado Ex2">
        <div class="modal-actions">
          <button class="save-button" @click="savePacientes">Salvar</button>
          <button class="cancel-button" @click="showModal = false">Cancelar</button>
        </div>
      </div>
    </div>

    <div v-if="showModalCalculo" class="modal">
      <div class="modal-content">
        <label for="nome">Exame 1:</label>
        <input type="text" disabled v-model="model.paciente.valorResultadoEx1" placeholder="Exame 1">
        <label for="sexo">Exame 2:</label>
        <input type="text" disabled v-model="model.paciente.valorResultadoEx2" placeholder="Exame 2">
        <div class="modal-actions">
          <button class="save-button" @click="calcular(model.paciente.valorResultadoEx1, model.paciente.valorResultadoEx2)">Calcular</button>
          <button class="cancel-button" @click="showModalCalculo = false">Cancelar</button>
        </div><br>
        <label for="idade">Resultado:</label><label for="result"> {{resultCalculo}}</label>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

const API_ROUTE = 'https://localhost:7245/';

export default {
  name: 'pacientes',
  data() {
    return {
      pacientes: [],
      resultCalculo: '',
      showModal: false,
      showModalCalculo: false,
      model: {
        paciente: {
          id: 0,
          nome: '',
          sexo: '',
          idade: 0,
          valorResultadoEx1: '',
          valorResultadoEx2: '',
        },
        newPaciente: {
          id: 0,
          nome: '',
          sexo: '',
          idade: 0,
          valorResultadoEx1: '',
          valorResultadoEx2: '',
        }
      },
      isValidNome: true,
      isValidSexo: true,
      isValidIdade: true
    }
  },
  mounted() {
    this.getPacientes();
  },
  methods: {
    getPacientes() {
      axios.get(API_ROUTE + 'ControleDePacientes/getpacientes')
        .then(res => {
          this.pacientes = res.data.obj;
        })
        .catch(error => {
          console.error('Error fetching pacientes:', error);
        });
    },

    modalCalculo(index){
      this.model.paciente = { ...this.pacientes[index] };
      this.showModalCalculo = true;
    },

    validarCampos() {
      this.isValidNome = !!this.model.paciente.nome.trim() ;
      
      this.isValidSexo = ['M', 'F'].includes(this.model.paciente.sexo.trim().toUpperCase());
      
      this.isValidIdade = !!this.model.paciente.idade;
    },

    calcular(ex1, ex2){
      // Não entendi muito bem o que deveria ser feito neste calculo
      this.resultCalculo = ex2 + ' com ' + ex1;
    },

    editPacientes(index) {
      this.model.paciente = { ...this.pacientes[index] };
      this.showModal = true;
    },

    addPaciente(){
      this.model.paciente = { ...this.model.newPaciente };
      this.showModal = true;
    },

    savePacientes() {
      if (!this.isValidNome || !this.isValidSexo || !this.isValidIdade) {
        alert('Todos os campos devem ser preenchidos!')
        return;
      }
      axios.post(API_ROUTE + 'ControleDePacientes/addeditpaciente', this.model.paciente).then(() => {this.getPacientes()});

      this.showModal = false;
    },

    deletePacientes(key) {
      axios.post(API_ROUTE + 'ControleDePacientes/deletepaciente?id=' + key).then(() => {this.getPacientes(), alert('Paciente excluido com sucesso!')}).then();

      this.showModal = false;
    },

  },
  watch: {
    'model.paciente.nome': function() {
      this.validarCampos();
    },
    'model.paciente.sexo': function() {
      this.validarCampos();
    },
    'model.paciente.idade': function() {
      this.validarCampos();
    }
  }
}
</script>

<style>
#app {
  max-width: 800px;
  margin: 0 auto;
}

h1 {
  text-align: center;
}

.error-message {
  color: red;
  font-size: 12px;
}

.add-button {
  margin-bottom: 20px;
  background-color: #4CAF50;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.pacientes-table {
  width: 100%;
  border-collapse: collapse;
}

.pacientes-table th, .pacientes-table td {
  padding: 8px;
  border: 1px solid #ddd;
}

.pacientes-table th {
  background-color: #f2f2f2;
}

.edit-button, .delete-button {
  background-color: #008CBA;
  color: white;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 5px;
}

.edit-button:hover, .delete-button:hover {
  background-color: #005f73;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 5px;
}

.modal input {
  margin-bottom: 10px;
  width: 100%;
  padding: 8px;
  box-sizing: border-box;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
}

.save-button, .cancel-button {
  background-color: #4CAF50;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-left: 5px;
}

.save-button:hover, .cancel-button:hover {
  background-color: #45a049;
}
</style>
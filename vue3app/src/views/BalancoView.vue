<template>
    <div class="about">
        <h1>Balanços</h1>
        <!-- Campos de entrada para o cadastro de balanços -->
        <p><label for="descricao">Descrição: </label><input id="descricao" type="text" v-model="balanco.descricao" />
        </p>
        <p><label for="dataHora">Data e Hora: </label><input id="dataHora" type="datetime-local"
                v-model="balanco.dataHora" /></p>
        <p><label for="valor">Valor: </label><input id="valor" type="number" v-model="balanco.valor" /></p>
        <button @click="incluir">Incluir</button>

        <!-- Campos de entrada para a consulta de balanços -->
        <h2>Consultar Balanços</h2>
        <p><label for="consultaDescricao">Descrição: </label><input id="consultaDescricao" type="text"
                v-model="consultaDescricao" />
        </p>
        <p><label for="consultaValor">Valor: </label><input id="consultaValor" type="number" v-model="consultaValor" />
        </p>
        <button @click="buscarBalancos">Consultar</button>
        <p>{{ erro }}</p>

        <!-- Exibição da lista de balanços ou mensagem de erro -->
        <div v-if="balancos.length === 0 && erro">
            <p>{{ erro }}</p>
        </div>
        <table v-else>
            <thead>
                <tr>
                    <th>Id</th>
                    <th>Descrição</th>
                    <th>Data e Hora</th>
                    <th>Valor</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="balanco in balancos" :key="balanco.id">
                    <td>{{ balanco.id }}</td>
                    <td>{{ balanco.descricao }}</td>
                    <td>{{ balanco.dataHora }}</td>
                    <td>{{ situacao(balanco.valor) }}</td>
                </tr>
            </tbody>
        </table>
        <button @click="limparConsulta">Mostrar Todos</button>
    </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import axios from 'axios';

// Definição dos dados do balanco
const balanco = ref({
    id: '',
    descricao: '',
    dataHora: '',
    valor: null
});

const consultaDescricao = ref('');
const consultaValor = ref<number | null>(null);

interface Balanco {
    id: string;
    descricao: string;
    dataHora: string;
    valor: number | null;
}

const balancos = ref<Balanco[]>([]);

const erro = ref('');

async function incluir() {
    erro.value = '';
    try {
        await axios.post('balanco', balanco.value);
        limparCampos(); // Limpar os campos após o cadastro
        buscarTodosBalancos(); // Atualizar a lista
    } catch (e) {
        erro.value = (e as Error).message;
    }
}

function limparCampos() {
    balanco.value = {
        id: '',
        descricao: '',
        dataHora: '',
        valor: null
    };
}

async function buscarTodosBalancos() {
    erro.value = '';
    try {
        const response = await axios.get('balanco');
        balancos.value = response.data;
    } catch (e) {
        erro.value = (e as Error).message;
    }
}

async function buscarBalancos() {
    erro.value = '';
    if (consultaDescricao.value && consultaValor.value !== null) {
        try {
            const url = `balanco/${consultaDescricao.value}/${consultaValor.value}`;
            const response = await axios.get(url);
            balancos.value = response.data;

            if (balancos.value.length === 0) {
                erro.value = 'Nenhum balanço foi encontrado para os critérios fornecidos';
            } else {
                erro.value = ''; // Limpa a mensagem de erro se houver resultados
            }
        } catch (e) {
            erro.value = (e as Error).message;
        }
    } else if (!consultaDescricao.value && !consultaValor.value) {
        buscarTodosBalancos();
    } else {
        erro.value = 'Por favor, preencha todos os campos para consulta.';
    }
}

function limparConsulta() {
    consultaDescricao.value = '';
    consultaValor.value = null;
    erro.value = '';
    buscarTodosBalancos();
}

function situacao(valor: number | null) {
    if (valor === null) {
        return ''; // ou outra mensagem apropriada para valores nulos
    } else {
        return valor >= 0 ? `${valor.toFixed(2)} C` : `${valor.toFixed(2)} D`;
    }
}


onMounted(() => {
    buscarTodosBalancos();
});
</script>
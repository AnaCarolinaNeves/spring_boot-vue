<template>
    <div class="about">
        <h1>Gerencie Vocábulos</h1>
        <!-- Campos de entrada para o cadastro de vocábulos -->
        <p><label for="termo">Termo: </label><input id="termo" type="text" v-model="vocabulo.termo" /></p>
        <p><label for="significado">Significado: </label><input id="significado" type="text"
                v-model="vocabulo.significado" /></p>
        <p><label for="versao">Versão: </label><input id="versao" type="number" v-model="vocabulo.versao" /></p>
        <p><label for="dataHoraCadastro">Data e Hora do Cadastro: </label><input id="dataHoraCadastro"
                type="datetime-local" v-model="vocabulo.dataHoraCadastro" /></p>
        <p><label for="dataHoraDesativacao">Situação: </label><input id="dataHoraDesativacao" type="datetime-local"
                v-model="vocabulo.dataHoraDesativacao" /></p>
        <button @click="incluir">Incluir</button>

        <!-- Campos de entrada para a consulta de vocábulos -->
        <h2>Consultar Vocábulo</h2>
        <p><label for="consultaTermo">Termo: </label><input id="consultaTermo" type="text" v-model="consultaTermo" />
        </p>
        <p><label for="consultaVersao">Versão: </label><input id="consultaVersao" type="number"
                v-model="consultaVersao" /></p>
        <button @click="buscarVocabulos">Consultar</button>
        <p>{{ erro }}</p>

        <!-- Exibição da lista de vocábulos ou mensagem de erro -->
        <div v-if="vocabulos.length === 0 && erro">
            <p>{{ erro }}</p>
        </div>
        <table v-else>
            <thead>
                <tr>
                    <th>Id</th>
                    <th>Termo</th>
                    <th>Significado</th>
                    <th>Versão</th>
                    <th>Data e Hora do Cadastro</th>
                    <th>Situação</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="vocabulo in vocabulos" :key="vocabulo.id">
                    <td>{{ vocabulo.id }}</td>
                    <td>{{ vocabulo.termo }}</td>
                    <td>{{ vocabulo.significado }}</td>
                    <td>{{ vocabulo.versao }}</td>
                    <td>{{ vocabulo.dataHoraCadastro }}</td>
                    <td>{{ situacao(vocabulo.dataHoraDesativacao) }}</td>
                </tr>
            </tbody>
        </table>
        <button @click="limparConsulta">Mostrar Todos</button>
    </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import axios from 'axios';

// Definição dos dados do vocábulo
const vocabulo = ref({
    id: '',
    termo: '',
    significado: '',
    versao: null,
    dataHoraCadastro: '',
    dataHoraDesativacao: '',
});

const consultaTermo = ref('');
const consultaVersao = ref<number | null>(null);

const vocabulos = ref([]);
const erro = ref('');

async function incluir() {
    erro.value = '';
    try {
        await axios.post('vocabulo', vocabulo.value);
        limparCampos(); // Limpar os campos após o cadastro
        buscarTodosVocabulos(); // Atualizar a lista
    } catch (e) {
        erro.value = (e as Error).message;
    }
}

function limparCampos() {
    vocabulo.value = {
        id: '',
        termo: '',
        significado: '',
        versao: null,
        dataHoraCadastro: '',
        dataHoraDesativacao: '',
    };
}

async function buscarTodosVocabulos() {
    erro.value = '';
    try {
        const response = await axios.get('vocabulo');
        vocabulos.value = response.data;
    } catch (e) {
        erro.value = (e as Error).message;
    }
}

async function buscarVocabulos() {
    erro.value = '';
    if (consultaTermo.value && consultaVersao.value !== null) {
        try {
            const url = `vocabulo/${consultaTermo.value}/${consultaVersao.value}`;
            const response = await axios.get(url);
            vocabulos.value = response.data;

            if (vocabulos.value.length === 0) {
                erro.value = 'Nenhum vocábulo foi encontrado para os critérios fornecidos';
            } else {
                erro.value = ''; // Limpa a mensagem de erro se houver resultados
            }
        } catch (e) {
            erro.value = (e as Error).message;
        }
    } else if (!consultaTermo.value && !consultaVersao.value) {
        buscarTodosVocabulos();
    } else {
        erro.value = 'Por favor, preencha todos os campos para consulta.';
    }
}

function limparConsulta() {
    consultaTermo.value = '';
    consultaVersao.value = null;
    erro.value = '';
    buscarTodosVocabulos();
}

function situacao(dataHoraDesativacao: string | null) {
    return dataHoraDesativacao ? 'Desativado' : 'Ativo';
}

onMounted(() => {
    buscarTodosVocabulos();
});
</script>
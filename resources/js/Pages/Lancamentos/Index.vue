<script setup>

import { ref, reactive, watch } from 'vue'
import { Head, useForm, Link, usePage, router} from '@inertiajs/vue3';
import AppLayout from '@/Layouts/AppLayout.vue';
import Tabela from '@/Components/Tabela.vue'
import Modal from '@/Components/Modal.vue'
import TextInput from '@/Components/TextInput.vue'
import SelectInput from '@/Components/SelectInput.vue'
import InputError from '@/Components/InputError.vue'
import InputLabel from '@/Components/InputLabel.vue'
import Messages from '@/Components/Messages.vue'
import Pagination from '@/Components/Pagination.vue';
import Filtro from '@/Components/Filtro.vue';

const props = defineProps({
    lancamentos: Object,
    categorias: Array,
    filtros: Object,
})

const filtros = ref({
    ano: props.filtros?.ano || '',
    mes: props.filtros?.mes || '',
    categoria_id: props.filtros?.categoria_id || '',
    tipo: props.filtros?.tipo || ''
});

const loading = ref(false);

const availableYears = [2025, 2024, 2023, 2022]; // Ajuste conforme seu backend
const meses = {
  1: 'Janeiro', 2: 'Fevereiro', 3: 'Março',
  4: 'Abril', 5: 'Maio', 6: 'Junho',
  7: 'Julho', 8: 'Agosto', 9: 'Setembro',
  10: 'Outubro', 11: 'Novembro', 12: 'Dezembro'
}

function aplicarFiltros() {
    
    router.get('/lancamentos', filtros.value, {
        preserveState: true,
        preserveScroll: true
    });  
    
}

const resetFilters = () => {
  filtros.value = {
    ano: new Date().getFullYear(),
    mes: '',
    categoria_id: '',
    tipo: ''
  }
  aplicarFiltros()
}

const estruturaInicialDoForm = {
  id: null,
  tipo: '',
  valor: 0,
  descricao: '',
  categoria_id: '',
  data: '',
  intervalo_meses: 0,
  selected_interval_option: '0',  
  custom_interval_value: 1,
  fim_da_recorrencia: '',
  esta_ativa: true,
};

const form = useForm({
    item: { ...estruturaInicialDoForm },
});

const showModalDelete = ref(false)
const showModalEdit = ref(false)

function closeModal(modal) {
    
    if (modal === 'edit') {
        showModalEdit.value = false
    }
    if (modal === 'delete') {
        showModalDelete.value = false
    }

    form.reset();
    form.clearErrors();
}

function openModal(modalRef, lancamento) {
    if (!lancamento || !lancamento.id) {
        console.error("Tentativa de abrir modal com lançamento inválido:", lancamento);
        return;
    }

    form.reset(); 

    form.item = {
        ...estruturaInicialDoForm, 
        ...lancamento,
        intervalo_meses: parseInt(lancamento.intervalo_meses) || 0,   
    };

    const currentInterval = form.item.intervalo_meses;

    if (currentInterval === 0) {
        form.item.selected_interval_option = '0';
        form.item.custom_interval_value = 1; // Reset para default
    } else if (currentInterval === 1) {
        form.item.selected_interval_option = '1';
        form.item.custom_interval_value = 1; // Reset para default
    } else if (currentInterval === 12) {
        form.item.selected_interval_option = '12';
        form.item.custom_interval_value = 1; // Reset para default
    } else if (currentInterval > 0) { // Qualquer outro valor numérico positivo
        form.item.selected_interval_option = 'custom';
        form.item.custom_interval_value = currentInterval;
    } else { // Fallback, embora já convertido para 0 acima
        form.item.selected_interval_option = '0';
        form.item.custom_interval_value = 1;
    }
    modalRef.value = true;
}


function handleIntervalOptionChange() {
    const option = form.item.selected_interval_option;
    if (option === 'custom') {
        if (!form.item.custom_interval_value || form.item.custom_interval_value < 1) {
            form.item.custom_interval_value = 1;
        }
        form.item.intervalo_meses = parseInt(form.item.custom_interval_value) || 1;
    } else {
        form.item.intervalo_meses = parseInt(option);
    }
}

function updateIntervaloMesesFromCustom() {
    if (form.item.selected_interval_option === 'custom') {
        form.item.intervalo_meses = parseInt(form.item.custom_interval_value) || 1;
    }
}

function onEdit(lancamento) {
    openModal(showModalEdit, lancamento)
}

function onDelete(lancamento) {
    openModal(showModalDelete, lancamento)
}

const fields = [
    { key: 'tipo', label: 'Tipo' },
    { key: 'descricao', label: 'Descrição' },
    { key: 'categoria.nome', label: 'Categoria' },
    { key: 'valor', label: 'Valor (R$)' },
    { key: 'data', label: 'Data Cadastro' },
    { key: 'intervalo_meses', label: 'Recorrência (meses)' },
];

const submit = () => {
   
    const { selected_interval_option, custom_interval_value, ...itemParaEnviar } = form.item;

    form.transform(() => itemParaEnviar).put(route('lancamentos.update', itemParaEnviar.id), {
            preserveScroll: true,
            onSuccess: () => closeModal('edit'),
        });
};

const deleteItem = () => {
    form.delete(route('lancamentos.destroy', form.item.id), {
        preserveScroll: true,
        onSuccess: () => closeModal('delete'),
    });
};

</script>

<template>
    <AppLayout title="Lançamentos">
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                Lançamentos
            </h2>
        </template>

        <Messages/>

        <div class="p-6">
            
            <Filtro :loading="loading" @update="aplicarFiltros" @reset="resetFilters">
                <template #default="{ onChange }">
                    <div>
                    <label class="block text-sm font-medium text-gray-700">Ano</label>
                    <select v-model="filtros.ano" @change="onChange" 
                            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500">
                        <option value="">Todos os anos</option>
                        <option v-for="year in availableYears" :key="year" :value="year">
                        {{ year }}
                        </option>
                    </select>
                    </div>

                    <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Mês</label>
                        <select v-model="filtros.mes" @change="onChange" class="w-full border border-gray-300 rounded-md px-3 py-2 focus:ring-blue-500 focus:border-blue-500">
                        <option value="">Todos os meses</option>
                        <option v-for="(nome, numero) in meses" :key="numero" :value="numero">{{ nome }}</option>
                        </select>
                    </div>

                    <div>
                    <label class="block text-sm font-medium text-gray-700">Categoria</label>
                    <select v-model="filtros.categoria_id" @change="onChange"
                            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500">
                        <option value="">Todas as categorias</option>
                        <option v-for="categoria in categorias" :key="categoria.id" :value="categoria.id">
                        {{ categoria.nome }}
                        </option>
                    </select>
                    </div>
                </template>
            </Filtro>

            <div class="flex justify-end gap-4 m-6">
                <Link :href="route('lancamentos.create')" class="inline-flex items-center px-5 py-2.5 rounded-xl bg-blue-600 text-white font-semibold shadow-md hover:bg-blue-700 transition">
                    <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 mr-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
                    </svg>
                    Novo Lançamento
                </Link>

                <Link :href="route('lancamentos.import')" class="inline-flex items-center px-5 py-2.5 rounded-xl bg-green-600 text-white font-semibold shadow-md hover:bg-green-700 transition">
                    <svg xmlns="http://www.w3.org/2000/svg" class="w-5 h-5 mr-2" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v16h16M4 4l16 16" />
                    </svg>
                    Importar Lançamentos
                </Link>
            </div>
            
            <Tabela :data="lancamentos.data" :fields="fields" @delete="onDelete" @edit="onEdit" />

            <Pagination :pagination="lancamentos" />
        </div>

        <!-- MODAL DE EDIÇÃO -->
        <Modal :show="showModalEdit" @close="() => closeModal('edit')">
            <template #default>
                <div class="p-6">
                    <h2 class="text-2xl font-bold mb-6 text-gray-800 flex items-center gap-2">
                        <svg class="w-6 h-6 text-blue-500" fill="none" stroke="currentColor" stroke-width="2"
                            viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                            <path stroke-linecap="round" stroke-linejoin="round"
                                d="M15.232 5.232l3.536 3.536m-2.036-2.036A9 9 0 1112 3a9 9 0 014.732 3.732zm0 0L21 3" />
                        </svg>
                        Editar Lançamento
                    </h2>
                    <form @submit.prevent="submit">
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
                            <!-- Tipo -->
                            <div>
                                <InputLabel for="edit-tipo" value="Tipo" :required="true" />
                                <select v-model="form.item.tipo" id="edit-tipo"
                                    class="border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 rounded-md shadow-sm mt-1 block w-full"
                                    required>
                                    <option value="" disabled>Selecione...</option>
                                    <option value="despesa">Despesa</option>
                                    <option value="receita">Receita</option>
                                </select>
                                <InputError class="mt-2" :message="form.errors.tipo" />
                            </div>
                            <!-- Valor -->
                            <div>
                                <InputLabel for="edit-valor" value="Valor" :required="true" />
                                <TextInput id="edit-valor" v-mask-decimal placeholder="0.00"
                                    v-model="form.item.valor" class="mt-1 block w-full" step="0.01"
                                    required />
                                <InputError class="mt-2" :message="form.errors.valor" />
                            </div>
                            <!-- Descrição -->
                            <div class="md:col-span-2">
                                <InputLabel for="edit-descricao" value="Descrição" :required="true" />
                                <TextInput id="edit-descricao" v-model="form.item.descricao" type="text"
                                    class="mt-1 block w-full" required />
                                <InputError class="mt-2" :message="form.errors.descricao" />
                            </div>
                            <!-- Categoria -->
                            <div>
                                <InputLabel for="edit-categoria" value="Categoria" :required="true" />
                                <SelectInput id="edit-categoria" v-model="form.item.categoria_id"
                                    required> <!-- Seu componente SelectInput -->
                                    <option value="">Sem categoria</option>
                                    <option v-for="categoria in categorias" :key="categoria.id" :value="categoria.id">{{
                                        categoria.nome }}
                                    </option>
                                </SelectInput>
                                <InputError class="mt-2" :message="form.errors.categoria_id" />
                            </div>
                            <!-- Data -->
                            <div>
                                <InputLabel for="edit-data" value="Data" :required="true" />
                                <TextInput id="edit-data" v-model="form.item.data" v-maska="'##/##/####'"
                                    placeholder="dd/mm/aaaa" class="mt-1 block w-full" required />
                                <InputError class="mt-2" :message="form.errors.data" />
                            </div>
                            
                            <!-- Recorrência (Select) -->
                            <div>
                                <InputLabel for="edit-selected_interval_option" value="Recorrência" :required="true" />
                                <select 
                                    v-model="form.item.selected_interval_option"
                                    id="edit-selected_interval_option"
                                    @change="handleIntervalOptionChange"
                                    class="border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 rounded-md shadow-sm mt-1 block w-full"
                                    required>
                                    <option value="0">Sem Recorrência</option>
                                    <option value="1">Mensal (1 mês)</option>
                                    <option value="12">Anual (12 meses)</option>
                                    <option value="custom">Personalizado</option>
                                </select>
                                <InputError class="mt-2" :message="form.errors.intervalo_meses" />
                            </div>

                            <!-- Intervalo Personalizado (meses) -->
                            <div v-if="form.item.selected_interval_option === 'custom'">
                                <InputLabel for="edit-custom_interval_value" value="Intervalo personalizado (meses)" :required="form.item.selected_interval_option === 'custom'" />
                                <TextInput 
                                    id="edit-custom_interval_value"
                                    v-model.number="form.item.custom_interval_value"
                                    type="number"
                                    min="1"
                                    class="mt-1 block w-full"
                                    placeholder="Ex: 3"
                                    :required="form.item.selected_interval_option === 'custom'"
                                    @input="updateIntervaloMesesFromCustom"
                                />
                                <!-- O erro ainda será em 'intervalo_meses' se a validação do backend falhar para o valor numérico -->
                                <InputError class="mt-2" :message="form.errors.intervalo_meses" />
                            </div>
                            
                            <!-- Fim da Recorrência -->
                            <div v-if="form.item.intervalo_meses && form.item.intervalo_meses > 0">
                                <InputLabel for="edit-fim_da_recorrencia" value="Fim da Recorrência" />
                                <TextInput id="edit-fim_da_recorrencia" v-model="form.item.fim_da_recorrencia"
                                    v-maska="'##/##/####'" placeholder="dd/mm/aaaa" class="mt-1 block w-full" />
                                <InputError class="mt-2" :message="form.errors.fim_da_recorrencia" />
                            </div>

                            <!-- Esta Ativa -->
                            <div class="flex items-center mt-4 md:col-span-2">
                                <input id="edit-esta_ativa" v-model="form.item.esta_ativa" type="checkbox"
                                    class="h-5 w-5 border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 rounded-md shadow-sm m-1 block" />
                                <InputLabel for="edit-esta_ativa" value="Está ativa?" class="ml-2 mb-0" />
                                <InputError class="mt-2" :message="form.errors.esta_ativa" />
                            </div>
                        </div>
                        <!-- Botões -->
                        <div class="mt-8 flex justify-end gap-3">
                            <button type="button" @click="() => closeModal('edit')"
                                class="px-5 py-2 rounded-xl bg-gray-200 text-gray-800 hover:bg-gray-300 font-semibold transition">
                                Cancelar
                            </button>
                            <button type="submit" :disabled="form.processing"
                                class="px-6 py-2 rounded-xl bg-blue-600 text-white font-bold hover:bg-blue-700 transition">
                                Salvar Alterações
                            </button>
                        </div>
                    </form>
                </div>
            </template>
        </Modal>

        <Modal :show="showModalDelete" @close="() => closeModal('delete')">
            <template #default>
                <div class="p-6">
                    <h3 class="text-lg font-bold mb-4 text-red-600">Confirmar Exclusão</h3>
                    <p class="mb-6">
                        Tem certeza que deseja excluir 
                        <strong>"{{ form.item?.descricao || 'este item' }}"</strong>?
                        Esta ação não pode ser desfeita.
                    </p>
                    <div class="flex justify-end gap-3">
                        <button type="button" @click="() => closeModal('delete')"
                            class="px-4 py-2 bg-gray-300 text-gray-700 rounded-lg hover:bg-gray-400 transition">
                            Cancelar
                        </button>
                        <button @click="deleteItem" 
                            class="px-4 py-2 bg-red-600 text-white rounded-lg hover:bg-red-700 transition">
                            Excluir Lançamento
                        </button>
                    </div>
                </div>
            </template>
        </Modal>
    </AppLayout>
</template>
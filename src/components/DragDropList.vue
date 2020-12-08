<template>
  <v-flex lg12 sm12 xs12 class="container d-flex align-stretch">
    <v-card
      class="mx-auto"
      width="40%"
      min-height="100%"
      tile
    >
      <div class="d-flex flex-column" style="height: 100%;">
        <v-card-title class="blue-grey lighten-4 black--text flex-grow-1">
          <h4 class="title font-weight-light mb-2">
            Itens Não Selecionados
          </h4>
        </v-card-title>
        <v-card-text class="d-flex " v-if="hasItemList">
          <h3 class="mr-auto d-flex align-self-center">Grupo 1</h3>
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn
                class="ma-2"
                @click="selectAllFunctionalityItems"
                v-on="on"
                x-small
                icon
              >
                <v-icon dark>mdi-plus-box-multiple</v-icon>
              </v-btn>
            </template>
            <span>Selecionar Todos</span>
          </v-tooltip>
        </v-card-text>

        <v-list-item v-if="hasItemList">
          <v-list-item-content style="height: 100%;">
            <div class="dashed-border" :style="hasUnselected ? 'border: none' : ''">
              <span v-if="!hasUnselected">Arraste itens para esta área para removê-los!</span>
              <draggable
                class="d-flex
                align-self-stretch flex-column"
                style="height: 100%;"
                v-model="functionality.items"
                group="functionality"
                @start="drag=true"
                @end="drag=false"
              >
                <template v-if="hasUnselected">
                  <v-chip class="mb-1 d-flex mr-auto" v-for="element in functionality.items" :key="element.id" color="gray">
                    <span>{{element.name}}</span>
                    <v-icon class="dark" small>mdi-close</v-icon>
                  </v-chip>
                </template>
              </draggable>
            </div>
          </v-list-item-content>
        </v-list-item>
        <v-list-item v-else />
      </div>
    </v-card>

    <div class="d-flex flex-column align-self-center">
      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <div class="d-flex flex-column align-self-center">
            <v-btn
              class="ma-2"
              @click="comfirm"
              v-on="on"
              small
              fab
              icon
            >
              <v-icon dark>mdi-check</v-icon>
            </v-btn>
          </div>
        </template>
        <span>Confirmar</span>
      </v-tooltip>
      
      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <v-btn
            class="ma-2"
            @click="clear"
            v-on="on"
            small
            fab
            icon
          >
            <v-icon dark>mdi-close</v-icon>
          </v-btn>
        </template>
        <span>Cancelar</span>
      </v-tooltip>
    </div>

    <v-card
      class="mx-auto"
      width="40%"
      min-height="100%"
      tile
    >
      <div class="d-flex flex-column" style="height: 100%;">
        <v-card-title class="blue-grey lighten-4 black--text flex-grow-1">
          <h4 class="title font-weight-light mb-2 d-flex">
            Itens Selecionados
          </h4>
        </v-card-title>
        <v-card-text class="d-flex" v-if="hasItemList">
          <h3 class="mr-auto d-flex align-self-center">Grupo 1</h3>
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn
                class="ma-2"
                @click="deselectAllFunctionalityItems"
                v-on="on"
                x-small
                icon
              >
                <v-icon dark>mdi-minus-box-multiple</v-icon>
              </v-btn>
            </template>
            <span>Deselecionar Todos</span>
          </v-tooltip>
        </v-card-text>
        <v-list-item v-if="hasItemList">
          <v-list-item-content style="height: 100%;">
            <div class="dashed-border" :style="hasSelected ? 'border: none' : ''">
              <span v-if="!hasSelected">Arraste itens para esta área para adicioná-los!</span>
              <draggable
                class="d-flex align-self-stretch flex-column"
                style="height: 100%;"
                v-model="functionality.selected"
                group="functionality"
                @start="drag=true"
                @end="drag=false"
              >
                <template v-if="hasSelected">
                  <v-chip class="mb-1 d-flex mr-auto" v-for="element in functionality.selected" :key="element.id" dark color="blue">
                    <span>{{element.name}}</span>
                    <v-icon class="dark" small>mdi-check</v-icon>
                  </v-chip>
                </template>
              </draggable>
            </div>
          </v-list-item-content>
        </v-list-item>
        <v-list-item v-else class="container">
          <v-expansion-panels accordion multiple>
            <v-expansion-panel v-for="item in control" :key="item.id">
              <div class="d-flex ml-5">
                <h5>{{ item.application.name }} - {{ item.profile.name }}</h5>
                <v-tooltip bottom class="mr-auto">
                  <template v-slot:activator="{ on }">
                    <v-btn
                      class="ma-2"
                      @click="removeLine(item)"
                      v-on="on"
                      x-small
                      icon
                    >
                      <v-icon dark>delete</v-icon>
                    </v-btn>
                  </template>
                  <span>Remover</span>
                </v-tooltip>
              </div>
              <v-divider></v-divider>
              <v-expansion-panel-header>
                Ver permissões
              </v-expansion-panel-header>
              <v-expansion-panel-content v-for="functionality in item.functionalityList" :key="functionality.id">
                <span>{{ functionality.name }}</span>
              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>
        </v-list-item>
      </div>
    </v-card>
  </v-flex>
</template>

<script>
import draggable from 'vuedraggable';

export default {
  components: {
    draggable,
    },
  name: 'DragDropList',
  data: () => ({
    id: null,
    model: {
      profileList: []
    },
    valid: true,
    validAuth: true,
    applicationSelected: { id: 1, name: 'Aplicação 1'},
    application: {
      items: [],
      isLoading: false
    },
    profileSelected: { id: 1, name: 'Perfil 1'},
    profile: {
      items: [],
      selecteds: [],
      isLoading: false
    },
    functionality: {
      items: [ 
        { id: 1, name: 'Item Numero 1' },
        { id: 2, name: 'Item Numero 2' },
        { id: 3, name: 'Item Numero 3' },
        { id: 4, name: 'Item Numero 4' },
        { id: 5, name: 'Item Numero 5' },
        { id: 6, name: 'Item Numero 6' },
        { id: 7, name: 'Item Numero 7' },
        { id: 8, name: 'Item Numero 8' },
        { id: 9, name: 'Item Numero 9' },
        { id: 10, name: 'Item Numero 10' },
      ],
      selected: []
    },
  }),
  computed: {
    hasItemList() {
      return this.hasSelected || this.hasUnselected
    },
    hasSelected() {
      return this.functionality.selected.length > 0
    },
    hasUnselected() {
      return this.functionality.items.length > 0
    }
  },
  methods: {
    selectAllFunctionalityItems () {
      this.functionality.selected = this.functionality.selected.concat(this.functionality.items)
      this.functionality.items = []
    },
    deselectAllFunctionalityItems () {
      this.functionality.items = this.functionality.items.concat(this.functionality.selected)
      this.functionality.selected = []
    },
    clear() {
      alert('Acao Cancelar')
    },
    comfirm() {
      alert('Acao Confirmar')
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.dashed-border {
  line-height: 45px;
  border: dashed 4px lightgray;
  border-radius: 20px;
  text-align: center;
  height: 100%;
  color: lightgray;
}
</style>

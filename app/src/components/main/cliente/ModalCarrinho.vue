<template>
  <div>
    <q-dialog v-model="mostraModal">
      <q-card class="card q-pa-lg">
        <q-card-section class="row q-pa-none q-mb-md">
          <q-btn
            class="absolute-top-right"
            icon="close"
            flat
            round
            dense
            v-close-popup
          />
          <div class="text-h4 text-grey-8">
            Meu Carrinho de Compra
            <q-icon name="shopping_cart" />
          </div>
        </q-card-section>

        <q-card-section class="row q-pa-md bg-blue-grey-4">
          <div class="text-h5 text-white text-weight-bold text-uppercase">
            Produtos
          </div>
          <q-separator color="white" class="q-mb-md" />
          <div class="col-12">
            <q-markup-table dense separator="cell" class="table-carrinho">
              <thead class="text-center sticky-head">
                <tr class="text-white bg-teal text-uppercase">
                  <th class="bold custom-font">Produto</th>
                  <th class="bold custom-font">Preço</th>
                  <th class="bold custom-font">Quantidade</th>
                  <th class="bold custom-font">Sub-total</th>
                </tr>
              </thead>
              <tbody class="text-center text-grey-8">
                <tr v-for="listaproduto in listaprodutos" :key="listaproduto">
                  <td class="text-center">
                    <q-list>
                      <q-item>
                        <q-item-section>
                          <q-item-label>
                            <q-btn
                              dense
                              round
                              color="red-5"
                              padding="xs"
                              @click="
                                removerProdutoCarrinho(listaproduto.produto)
                              "
                            >
                              <q-icon name="close" size="1em" />
                            </q-btn>
                            <q-avatar class="shadow-4 q-mx-md">
                              <q-img
                                :src="listaproduto.produto.imgUrl"
                                spinner-color="teal"
                                spinner-size="50px"
                                :thickness="5"
                                transition="jump-up"
                                :ratio="1"
                              />
                            </q-avatar>
                            <span class="text-subtitle2 ellipsis">
                              {{ listaproduto.produto.descricao }}
                            </span>
                          </q-item-label>
                        </q-item-section>
                      </q-item>
                    </q-list>
                  </td>
                  <td class="custom-font">
                    R$ {{ listaproduto.produto.preco }}
                  </td>
                  <td>
                    <q-input
                      ref="inputQtd"
                      rounded
                      outlined
                      dense
                      v-model.number="listaproduto.quantidade"
                      color="blue-grey-4"
                      type="number"
                      input-class="text-subtitle1 text-grey-8 text-center"
                      :rules="[
                        (val) =>
                          (val >= 1 && val <= produto.estoque_atual) ||
                          `Valor inválido!`,
                      ]"
                      class="q-pb-none"
                    >
                      <template v-slot:before>
                        <q-btn
                          round
                          dense
                          color="btn-quantidade"
                          icon="remove"
                          @click="menosQtdProduto"
                        />
                      </template>
                      <template v-slot:after>
                        <q-btn
                          round
                          dense
                          color="btn-quantidade"
                          icon="add"
                          @click="maisQtdProduto"
                        />
                      </template>
                    </q-input>
                  </td>
                  <td class="custom-font">R$ {{ listaproduto.total }}</td>
                </tr>
              </tbody>
            </q-markup-table>
          </div>
          <div class="col-12">
            <q-btn-group spread class="rounded-buttons">
              <q-btn
                color="blue-grey-3"
                icon="mdi-broom"
                label="Limpar Carrinho"
              />
              <q-btn color="orange-4" icon="mdi-currency-usd">
                <div class="custom-font">{{ cliente.carrinho.total }}</div>
              </q-btn>
              <q-btn
                color="green-5"
                icon="mdi-cart-check"
                label="Finalizar Compra"
                @click="finalizarCompra()"
              />
            </q-btn-group>
          </div>
        </q-card-section>
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
export default {
  props: ["mostraModal"],

  data() {
    return {
      quantidade: 0,
      listaprodutos: {},
      cliente: {},
    };
  },

  created() {
    window.setTimeout(() => {
      let cliente = this.$store.state.cliente;
      this.cliente = cliente;
      this.listaprodutos = this.$store.state.cliente.listaprodutos;
    }, 1000);
    // this.$http
    //   .get(`cliente/${cliente.id}/listacarrinho`)
    //   .then((res) => res.json())
    //   .then((listaprodutos) => {
    //     this.listaprodutos = listaprodutos;
    //     this.listaprodutos.forEach((listaproduto) => {
    //       listaproduto.produto.imgUrl = `${this.$http.options.root}/empresa/${listaproduto.produto.empresa_id}/images/produto/${listaproduto.produto.id}`;
    //     });
    //   }),
    //   (err) => {
    //     console.log(err);
    //   };
  },

  methods: {
    removerProdutoCarrinho(produto) {
      let cliente = this.$store.state.cliente;
      this.$http
        .delete(`cliente/${cliente.id}/listacarrinho/${produto.id}`)
        .then(
          (res) => {
            res.json();
            let indice = this.listaprodutos.findIndex(
              (x) => x.produto.id == produto.id
            );
            this.listaprodutos = [
              ...this.listaprodutos.slice(0, indice),
              ...this.listaprodutos.slice(indice + 1),
            ];
            this.successNotify(
              `Sucesso ao remover o produto ${produto.descricao} do carrinho!`
            );
          },
          (err) => {
            console.log(err);
            this.errorNotify(
              `Erro ao remover o produto ${produto.descricao} do carrinho!`
            );
          }
        );
    },

    finalizarCompra() {
      let cliente = this.$store.state.cliente;
      this.$http
        .post(`cliente/${cliente.id}/carrinho/${cliente.carrinho.id}`)
        .then(
          (res) => {
            res.json();
            cliente.listaprodutos = "";
            cliente.carrinho.qtd_itens = 0;
            cliente.carrinho.total = 0;
            this.listaprodutos = "";
            this.successNotify(`Compra realizada com sucesso!`);
          },
          (err) => {
            console.log(err);
            this.errorNotify(`Erro ao finalizar compra!`);
          }
        );
    },

    successNotify(msg) {
      this.$q.notify({
        progress: true,
        position: "top",
        type: "positive",
        message: msg,
      });
    },
    errorNotify(msg) {
      this.$q.notify({
        progress: true,
        position: "top",
        type: "negative",
        message: msg,
      });
    },
  },

  watch: {
    mostraModal: function () {
      if (!this.mostraModal) {
        this.$emit("fecharModal", this.mostraModal);
      }
    },
  },
};
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Oswald&display=swap");

.card {
  width: 80vw;
  max-width: 100vw;
  border-radius: 20px;
}

.bold {
  font-weight: bold !important;
  font-size: 1em;
}

.bg-btn-quantidade {
  background: #8abbbf;
}

.custom-font {
  font-family: "Oswald", sans-serif;
}

.table-carrinho {
  border-radius: 20px 20px 0px 0px;
  max-height: 45vh !important;
  overflow-y: scroll;
  overflow-x: auto;
}

.rounded-buttons {
  border-radius: 0px 0px 20px 20px;
}
</style>

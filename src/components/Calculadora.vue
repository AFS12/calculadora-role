<template>
  <v-container>
    <v-row>
      <v-col cols="4">
        <v-row>
          <v-col cols="3">
            <v-btn
              color="blue"
              dark
              @click="
                persons.push({
                  id: `${generateId('person')}`,
                  name: '',
                  toReceive: 0,
                  toPay: 0,
                  notConsume: [],
                })
              "
            >
              Adicionar pessoa
            </v-btn>
          </v-col>
        </v-row>
        <v-row
          class="text-center"
          v-for="person in persons"
          :key="person.id"
          dense
        >
          <v-col cols="6">
            <v-text-field
              label="Nome"
              v-model="person.name"
              @click:append="persons.splice(person.id, 1)"
              append-icon="mdi-close"
              outlined
              color
            />
          </v-col>
          <v-col cols="6">
            <v-select
              color
              v-model="person.notConsume"
              :items="items"
              item-text="name"
              item-value="id"
              outlined
              label="Não consumiu esses itens"
              multiple
              small-chips
              @change="calculatePaymentsWithNoPay()"
            ></v-select>
          </v-col>
        </v-row>
      </v-col>
      <v-divider vertical color="gray"></v-divider>
      <v-divider vertical color="gray"></v-divider>
      <v-divider vertical color="gray"></v-divider>
      <v-divider vertical color="gray"></v-divider>
      <v-col cols="8">
        <v-row>
          <v-col cols="3">
            <v-btn
              color="blue"
              dark
              @click="
                items.push({
                  id: `${generateId('item')}`,
                  name: '',
                  price: 0,
                  notConsumePersonsAndValues: { persons: [], value: 0 },
                })
              "
            >
              Adicionar itens comprados
            </v-btn>
          </v-col>
        </v-row>
        <v-row class="text-center" v-for="item in items" :key="item.id" dense>
          <v-col cols="4">
            <v-text-field
              label="Nome"
              v-model="item.name"
              @click:append="items.splice(item.id, 1), eraseNotConsume()"
              append-icon="mdi-close"
              outlined
              color
            />
          </v-col>
          <v-col cols="4">
            <v-text-field
              label="Preço"
              placeholder="0"
              type="number"
              v-model="item.price"
              @change="stackItemsPrice(), calculatePayments()"
              outlined
              color
            />
          </v-col>
          <v-col cols="4">
            <v-select
              color
              v-model="item.buyer"
              :items="persons"
              item-text="name"
              item-value="id"
              outlined
              label="Quem comprou"
              @change="calculatePayments()"
            ></v-select>
          </v-col>
        </v-row>
      </v-col>
    </v-row>
    <h1>Valor total: {{ totalAmount }}</h1>
    <v-row>
      <v-col>
        <v-simple-table>
          <template v-slot:default>
            <thead>
              <tr>
                <th class="text-left">Nome</th>
                <th class="text-left">Recebe</th>
                <th class="text-left">Paga</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="person in persons" :key="person.id">
                <td>{{ person.name }}</td>
                <td>{{ person.toReceive.toFixed(2) }}</td>
                <td>{{ person.toPay.toFixed(2) }}</td>
              </tr>
            </tbody>
          </template>
        </v-simple-table>
      </v-col>
    </v-row>
    <!-- {{ persons }}
    <br />
    {{ items }} -->
  </v-container>
</template>

<script>
export default {
  name: "Calculadora",

  data: () => ({
    personsIdCounter: 0,
    itemsIdCounter: 0,
    totalAmount: 0,
    reCalculate: false,
    persons: [
      {
        id: 0,
        name: "",
        toReceive: 0,
        toPay: 0,
        notConsume: [],
      },
    ],
    items: [
      {
        id: 0,
        name: "",
        price: 0,
        buyer: null,
        notConsumePersonsAndValues: { persons: [], value: 0 },
      },
    ],
  }),

  methods: {
    generateId(type) {
      if (type == "person") {
        this.personsIdCounter++;
        return this.personsIdCounter;
      } else {
        this.itemsIdCounter++;
        return this.itemsIdCounter;
      }
    },
    stackItemsPrice() {
      this.totalAmount = 0;
      this.items.forEach((item) => {
        this.totalAmount += Number(item.price);
      });
    },
    calculatePayments() {
      this.eraseAcumulated();
      this.persons.forEach((person) => {
        let isBuyer = false;

        person.toReceive = 0;
        this.items.forEach((item) => {
          if (person.id == item.buyer) {
            person.toReceive += Number(item.price);
            isBuyer = true;
          }
        });

        if (isBuyer) {
          person.toReceive =
            person.toReceive - this.totalAmount / this.persons.length;
          person.toReceive < 0 ? (person.toReceive = 0) : "";
        }
        person.toPay =
          this.totalAmount / this.persons.length - person.toReceive;
        person.toPay < 0 ? (person.toPay = 0) : "";
      });
    },
    calculatePaymentsWithNoPay() {
      this.calculatePayments();

      this.items.forEach((item) => {
        let itemPriceDivided = item.price / this.persons.length;
        let accumulate = 0;
        let personsNotPayThis = 0;

        this.persons.forEach((person) => {
          if (person.notConsume.includes(item.id)) {
            accumulate += itemPriceDivided;

            if (person.toPay > 0) {
              person.toPay -= itemPriceDivided;
              if (person.toPay < 0) {
                person.toReceive += person.toPay * -1;
                person.toPay = 0;
              }
            } else {
              person.toReceive += itemPriceDivided;
            }

            personsNotPayThis++;
          }
        });
        this.persons.forEach((person) => {
          if (!person.notConsume.includes(item.id)) {
            if (person.toReceive > 0) {
              person.toReceive -=
                accumulate / (this.persons.length - personsNotPayThis);
              if (person.toReceive < 0) {
                person.toPay += person.toReceive * -1;
                person.toReceive = 0;
              }
            } else {
              person.toPay +=
                accumulate / (this.persons.length - personsNotPayThis);
            }
          }
        });
      });
    },
    getItemPrice(id) {
      this.items.forEach((item) => {
        if (id == item.id) {
          return item.price;
        }
      });
    },

    pay(person) {
      return this.totalAmount / this.persons.length - person.toReceive;
    },
    eraseNotConsume() {
      this.persons.forEach((person) => {
        person.notConsume = [];
      });
      this.eraseAcumulated();
    },
    eraseAcumulated() {
      this.items.forEach((item) => {
        item.notConsumePersonsAndValues.value = 0;
        item.notConsumePersonsAndValues.persons = [];
      });
    },
  },
};
</script>

<template>
  <div>
    <div class="container">
      <div class="row">
        <div class="col">
          <div class="card bg-light border-info w-100">
            <h4 class="card-header">
              {{ titulo }}
            </h4>
            <ul class="list-group list-group-flush">
              <li class="list-group-item">          
                <input type="text" v-model="entrada" class="form-control" v-bind:class="{ 'is-valid': valido, 'is-invalid': entrada && !valido }" v-bind:placeholder="instrucciones" data-toggle="tooltip" data-placement="top" title="Tooltip on top">
              </li>
              <li class="list-group-item" v-if="valido">          
                <h4>
                  <span v-for="n in nahuatl" :key="n.numero">
                    {{ n.texto }}
                  </span>
                </h4>
                <!--{{ $data }}-->
              </li>
              <li class="list-group-item" v-if="!valido && entrada">
                {{ instrucciones }}
              </li>
            </ul>
            <table class="table table-striped mb-0" v-if="valido">
              <thead>
                <tr>
                  <th>Texto</th>
                  <th>Cálculo</th>
                  <th>Cantidad</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="n of nahuatl" :key="n.numero">
                  <td>{{ n.texto | textonumero }}</td>
                  <td>{{ n.numero | calculo }}</td>
                  <td>{{ n.numero }}</td>
                </tr>
                <tr v-if="entrada > 20">
                  <th colspan="2" class='text-center'>Total</th>
                  <td>{{ entrada }}</td>
                </tr>
              </tbody>
            </table>
            <small class="card-footer">
              {{ descripcion }}
            </small>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import db from "./../Database";
let auxiliaresRef = db.ref("auxiliares");
let principalesRef = db.ref("principales");

export default {
  name: "Tlapohuali",
  firebase: {
    auxiliares: {
      source: auxiliaresRef
    },
    principales: {
      source: principalesRef
    }
  },
  data() {
    return {
      titulo: "Tlapohuali",
      entrada: "",
      resta: Number,
      nahuatl: [],
      contador: Number,
      union: 'ipan',
      fecha: new Date().getFullYear(),
      descripcion: "Traductor de números arábigos al sistema de numeración náhuatl"
    };
  },
  computed: {
    valido: function() {
      return /^([1-9][0-9]?[0-9]?[0-9]?[0-9]?|1[0-5][0-9][0-9][0-9][0-9]|16[0-7][0-9][0-9][0-9])$/.test(
        this.entrada
      );
    },
    instrucciones: function() {
      return 'Escribe un número entre 1 y 167999 (únicamente dígitos, por ejemplo '+this.fecha+')';
    }
  },
  watch: {
    entrada: function() {
      if (this.valido) {
        this.nahuatl = [];
        this.contador = 0;
        this.resta = parseInt(this.entrada, 10);
        this.contar(this.resta);
        this.resta = parseInt(this.entrada, 10);
        this.traducir(this.resta);        
      } else {
        this.nahuatl = [];
      }
    }
  },
  methods: {
    traducir(numero) {
      const that = this;
      var principal = this.principales.filter(function(item) {
        return item.numero === numero;
      })[0];
      var auxiliar = this.auxiliares.filter(function(item) {
        return item.numero === numero;
      })[0];      
      if (principal === undefined) {
        for (let num of this.principales) {
          if (Math.floor(numero / Number(num.numero)) === 1) {
            this.traducir(num.numero);
            this.resta = this.resta - Number(num.numero);
            if (this.resta > 0) {
              this.traducir(this.resta);
            }
            break;
          }
        }
      } else {
        let texto = principal.texto;
        if (this.nahuatl.length) {
          if (this.contador >= 4) {
            texto = this.union + " " + principal.texto;
          }
          if (numero < 20) {
            texto = auxiliar.texto;
            if (this.contador >= 4) {
              texto = this.union + " " + auxiliar.texto;
            }
          }
        }
        let cantidad = numero;
        if (numero >= 20 && numero < 400) {
          cantidad = numero / 20;
        } else if (numero >= 400 && numero < 8000) {
          cantidad = numero / 400;
        } else if (numero >= 8000) {
          cantidad = numero / 8000;
        }
        this.nahuatl.push({ numero: numero, texto: texto, cantidad: cantidad });
        this.resta = this.resta - numero;
        if (this.resta > 0) {
          this.traducir(this.resta);
        }
      }
    },
    contar(numero) {
      const simples = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 15];
      var principal = this.principales.filter(function(item) {
        return item.numero === numero;
      })[0];      
      if (principal === undefined) {
        for (let num of this.principales){
          if (Math.floor(numero / Number(num.numero)) === 1) {
            this.contar(num.numero);
            this.resta = this.resta - Number(num.numero);
            if (this.resta > 0) {
              this.contar(this.resta);
            }
            break;
          }
        }
      } else {
        let con = 2;
        if (simples.indexOf(numero) > -1) {
          con = 1;
        }
        this.contador = this.contador + con;
        this.resta = this.resta - numero;
        if (this.resta > 0) {
          this.contar(this.resta);
        }        
      }
    }    
  },
  filters: {
    calculo: function (value) {
      if (!value) {
          return value;
      }
      let calculo = value;
      if (parseInt(value, 10) >= 20 && parseInt(value, 10) < 400) {
          calculo = (parseInt(value, 10) / 20) + ' X 20';
      } else if (parseInt(value, 10) >= 400 && parseInt(value, 10) < 8000) {
          calculo = (parseInt(value, 10) / 400) + ' X 400';
      } else if (parseInt(value, 10) >= 8000) {
          calculo = (parseInt(value, 10) / 8000) + ' X 8000';
      }
      return calculo;
    },
    textonumero: function(value) {
      if (!value) {
        return value;
      }
      return value.replace('ipan ', '');
    }
  }
};
</script>

<style scoped>

</style>

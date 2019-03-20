<template lang="pug">
  //v-layout(column justify-center align-center)
  v-layout(justify-center)
    v-flex(xs12 sm10)
      v-snackbar(v-model="snackbar.estado" top right :timeout="snackbar.timeout" :color="snackbar.color")
        |  {{ snackbar.texto }}
        v-btn(color="white" flat @click="snackbar.estado = false") Cerrar
      v-dialog(v-model="loading.estado" hide-overlay persistent width="400")
        v-card(:color="loading.color" dark)
          v-card-text {{ loading.texto }}
            v-progress-linear(indeterminate color="white" class="mb-0")
            span {{ 'Progreso %' + loading.carga }}
      v-card(color="grey lighten-4")
        v-card-title
          span.title Venta - &nbsp;
          v-icon(left color = "black") fab fa-cc-visa
          v-icon(left color = "black") fab fa-cc-mastercard
          v-icon(left color = "black") fab fa-cc-jcb
          v-icon(left color = "black") fab fa-cc-diners-club
          v-icon(left color = "black") fab fa-cc-amex
        v-divider
        v-card-text
          v-container
            v-layout
              v-flex(xs12 sm6 md3)
                v-text-field(type="number" label = "Importe" prepend-icon = "fas fa-dollar-sign" 
                :color = "$store.state.Colores.Header" autofocus hint = "Ejemplo: $25,50" persistent-hint
                v-model = "venta.importe" @keyup.enter = "Comprar")
              v-flex(xs12 sm6 md3)
                v-select(label = "Cuotas" prepend-icon = "fas fa-list-ol" 
                :color = "$store.state.Colores.Header" :items = "venta.cuotas" v-model = "venta.cuota"
                )
              v-flex(xs12 sm6 md6)
                v-text-field(type="text" label = "Factura" prepend-icon = "fas fa-file-invoice-dollar" 
                :color = "$store.state.Colores.Header" hint = "Ejemplo: 012100001525" persistent-hint 
                v-model = "venta.factura" @keyup.enter = "Comprar")
        v-alert(v-model="alerta.estado" dismissible :type="alerta.type") {{ alerta.texto }}
</template>
<script>
export default {
  data() {
    return {
      alerta: {
        estado: false,
        type: 'error',
        texto: ''
      },
      loading: {
        estado: false,
        texto: 'Se está interactuando con el POSNET',
        carga: 0,
        color: 'info'
      },
      snackbar: {
        estado: false,
        texto: '',
        color: '',
        timeout: 6000
      },
      venta: {
        importe: 0,
        cuotas: [1, 3, 6, 9, 12, 18],
        cuota: 1,
        factura: null
      }
    }
  },
  methods: {
    SetLoading(estado = false, texto = '', carga = 0, color = 'info') {
      this.loading.estado = estado
      this.loading.texto = texto
      this.loading.carga = carga
      this.loading.color = color
    },
    SetAlerta(texto, tipo, estado) {
      this.alerta.texto = texto
      this.alerta.type = tipo
      this.alerta.estado = estado
    },
    SetSnackBar(texto, color, estado) {
      this.snackbar.texto = texto
      this.snackbar.color = color
      this.snackbar.estado = estado
    },
    ConfirmarTransaccion() {
      const res = this.$axios.$post(
        'http://127.0.0.1:8280',
        {
          TransactionRequestType: 'ConfirmTransaction'
        },
        {
          headers: {
            'Content-Type': 'application/json'
          }
        }
      )
      return res
    },
    GetCard() {
      const res = this.$axios.$post(
        'http://127.0.0.1:8280',
        {
          TransactionRequestType: 'GetCard',
          TransactionType: 'Buy',
          ReadExpiration: '30'
        },
        {
          headers: {
            'Content-Type': 'application/json'
          }
        }
      )
      return res
    },
    Buy(importe, cuota, factura) {
      const res = this.$axios.$post(
        'http://127.0.0.1:8280',
        {
          TransactionRequestType: 'Buy',
          TransactionResolutionMode: 'Online',
          TransactionCurrencyCode: '032',
          TransactionAmount: importe,
          ReceiptNumber: `${factura}`,
          TransactionInstallments: cuota
        },
        {
          headers: {
            'Content-Type': 'application/json'
          }
        }
      )
      return res
    },
    Comprar() {
      if (
        this.venta.importe == '' ||
        this.venta.cuota == '' ||
        this.venta.factura == ''
      ) {
        this.SetSnackBar('Por favor, llená todos los campos', 'error', true)
      } else {
        this.SetLoading(true, 'Por favor, inserte tarjeta en el POSNET.', 25)
        const res = this.GetCard()
        res
          .then(res => {
            if (res.ResultMessage == 'CANCELADO') {
              this.SetLoading()
              this.SetSnackBar('Cancelado!', 'info', true)
              this.SetAlerta('Operacion cancelada por usuario', 'info', true)
            } else if (res.ResultMessage == 'No response from the device.') {
              this.SetLoading()
              this.SetSnackBar('Pinpad error', 'error', true)
              this.SetAlerta(
                'Ocurrio un error con el pinpad. Verifique si esta conectado y si está iniciado el servicio',
                'error',
                true
              )
            } else {
              this.SetLoading(
                true,
                'Espere, interactuando con la terminal.',
                75,
                'warning'
              )
              this.Buy(
                this.venta.importe,
                parseInt(this.venta.cuota),
                this.venta.factura
              ).then(res => {
                console.log(res)
                if (res.ResponseActions == 'Approve') {
                  this.SetSnackBar('Aprobada!', 'success', true)
                  this.SetAlerta(
                    'Transaccion aprobada. Autorizacion: ' +
                      res.HostAuthorizationCode +
                      ' | Tarjeta N°: ' +
                      res.CardNumber +
                      ' | Emisora: ' +
                      res.PaymentMethodDescription +
                      ' | Ingreso: ' +
                      res.CardInputModeDescription +
                      ' | Cupon/Lote: ' +
                      res.HostTicketNumber +
                      '/' +
                      res.HostBatchNumber +
                      ' | Importe: $' +
                      res.TransactionAmount +
                      ' | Fecha: ' +
                      res.TransactionDate +
                      ' ' +
                      res.TransactionTime +
                      ' | Ticket: ' +
                      res.ReceiptNumber,
                    'success',
                    true
                  )
                } else {
                  this.SetSnackBar('Problem', 'error', true)
                  this.SetAlerta(
                    'Cod: ' +
                      res.HostResultCode +
                      ' | MSG: ' +
                      res.HostResultMessage,
                    'error',
                    true
                  )
                }
                this.SetLoading()
                this.ConfirmarTransaccion()
              })
            }
          })
          .catch(err => {
            console.log(err)
            this.SetLoading()
            this.SetSnackBar(
              'El POSNET no está iniciado o no está instalado.',
              'error',
              true
            )
          })
      }
    }
  }
}
</script>

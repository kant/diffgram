<template>
  <div v-cloak>

    <!-- Careful, still need this to get relevant account
        but displaying info is optional,
        ie in case of order screen -->

    <v_account_info :show_account_info="show_account_info"
                    @account="account = $event">
    </v_account_info>


        </v-btn>


        <v-alert type="success"
                 :value="stripe_success">
          Card saved.
        </v-alert>

        <v-alert type="info"
                 outlined
                 :value="account.payment_method_on_file == true">
          Card on file.
        </v-alert>

        <!-- Hard to use normal loading thing since
       stripe wants to have the element loaded
       in the DOM-->

        <v-container v-if=" account.payment_method_on_file != true &&
                            !stripe_success">

          <form method="post" id="payment-form">
            <div class="form-row">
              <label for="card-element">
                Credit or debit card
              </label>
              <div id="card-element">
                <!-- A Stripe Element will be inserted here. -->
              </div>

              <!-- Used to display form errors. -->
              <div id="card-errors" role="alert"></div>
            </div>

          </form>
          <v-flex class="pa-3">

            <v-btn color="primary"
                   @click="createToken"
                   :disabled="loading">
              Add Card
            </v-btn>

          </v-flex>

          <v_error_multiple :error="error">
          </v_error_multiple>

        </v-container>

        <!-- Disable for Jan release -->
        <!--
        <v-btn color="primary"
               @click="stripe_charge_account">
          Charge
        </v-btn>
        -->




  </div>
</template>

<script lang="ts">

  import axios from 'axios';

  import Vue from "vue"; export default Vue.extend( {
    name: 'credit_card',
    props: {
      'show_account_info' : {
        default: true
      }
    },
    data() {
      return {
        loading: true,

        card: null,
        elements: null,

        stripe_success: false,

        account: {
          id: 1,
          payment_method_on_file: false
        },

        error: {

        },


        stripe: null,


        style : {
          base: {
            color: '#32325d',
            fontFamily: '"Helvetica Neue", Helvetica, sans-serif',
            fontSmoothing: 'antialiased',
            fontSize: '20px',
            '::placeholder': {
              color: '#aab7c4'
            }
          },
          invalid: {
            color: '#fa755a',
            iconColor: '#fa755a'
          }
        }

      }
    },

    computed: {

    },
    mounted() {

      let stripe_script = document.createElement('script')
      stripe_script.setAttribute('src', 'https://js.stripe.com/v3/')
      document.head.appendChild(stripe_script)

      var self = this

      // Test card
      //  4242 4242 4242 4242

      setTimeout(function () {

        if (self.account.payment_method_on_file != true) {



          // @ts-ignore
          //self.stripe = Stripe('pk_test_704Rwhx7wXhQUoPTfuqiN55i')
          // @ts-ignore
          self.stripe = Stripe('pk_live_QjJ0c5QXD4T61JLbNWN3m11F')
          self.elements = self.stripe.elements();

          self.card = self.elements.create('card', { style: self.style });

          self.card.mount('#card-element');

          // Handle real-time validation errors from the card Element.
          self.card.addEventListener('change', function (event) {
            var displayError = document.getElementById('card-errors');
            if (event.error) {
              displayError.textContent = event.error.message;
            } else {
              displayError.textContent = '';
            }
          });

          self.form = document.getElementById('payment-form');
        }

        self.loading = false

      }, 1000)



    },

    destroyed() {

    },

    created() {


    },

    methods: {
      createToken: function () {
        this.stripeCheck = true;

        // https://stripe.com/docs/stripe-js/reference#stripe-create-token

        this.stripe.createToken(this.card).then(result => {
          if (result.error) {
            // Inform the user if there was an error.
            var errorElement = document.getElementById('card-errors');
            errorElement.textContent = result.error.message;
          } else {
            // Send the token to your server.
            this.stripe_token_to_account(result.token);
          }
        });

      },

      stripe_token_to_account: function (token) {

        this.loading = true;
        this.error = {}

        axios.post(`/api/v1/project/${String(this.$store.state.project.current.project_string_id)}/account/billing/stripe/token`, {

            'account_id': this.account.id,
            'token': token

        }).then(response => {

          if (response.data.log.success == true) {
            this.stripe_success = true

          } else {

            this.error = response.data.log.error
          }

          this.loading = false


        })
          .catch(error => {
            this.error = this.$route_api_errors(error)
            //console.log(error);
            this.loading = false
          });
      },

      stripe_charge_account: function () {

        this.loading = true;

        // reset error
        this.error = {}

        axios.post('/api/v1/account/billing/stripe/charge', {

          'account_id': this.account.id

        }).then(response => {


          if (response.data.log.success == true) {

          } else {
            this.error = response.data.log.error
          }

          this.loading = false


        })
          .catch(error => {
            this.error = this.$route_api_errors(error)
            //console.log(error);
            this.loading = false
          });
      }
    }
  }

) </script>

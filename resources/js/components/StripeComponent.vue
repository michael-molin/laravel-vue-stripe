<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header">Componente Stripe Montato</div>
                    <p>Per pagamento a buon fine: 4242 4242 4242 4242</p>
                    <p>Per pagamento rifiutato: 4000 0000 0000 9995</p>
                    <div class="card-body">
                        <Label for="card-holder-name"> Nome Titolare Carta: </Label>
                        <input id="card-holder-name" type="text">
                        <hr>
                        <div id="card" ref="card"></div>
                        <button id="card-button" @click="toPayment" :disabled="isDisable"> 
                            <i class="fab fa-cc-stripe" v-if="isPay"></i>
                            <i class="fas fa-spinner" id="loading" v-if="isLoading"></i>
                            <i class="fas fa-check-circle" v-if="isOk"></i>
                            <i class="fas fa-times" v-if="isError"></i>
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                spk:'pk_test_51HMa46FUEyYvJlZwxKqaMFLFSJ8cci1OWaA0Adeavbtm2M8AEkShK10Hlijudr0m7NFYRCllxhIumqWa6Ib0i9Az00pEC788en',
                stripe: '',
                card: '',
                isPay: true,
                isLoading : false,
                isOk : false,
                isError : false,
                isDisable : false
            }
        },
        mounted() {    
            var self=this;
            self.stripe= Stripe(self.spk);
            self.card = self.stripe.elements().create('card', {
                hidePostalCode : true,
                style : {
                    base : {
                        color: '#636b6f'
                    }
                    
                }
            });
            self.card.mount(self.$refs.card);
        },

        methods : {

            async toPayment() {
                this.isDisable=true;
                this.isPay= false;
                this.isLoading= true;
                const cardButton = document.getElementById('card-button');
                const cardHolderName = document.getElementById('card-holder-name');
                const { paymentMethod, error } = await this.stripe.createPaymentMethod({
                    type: 'card',
                    card: this.card,
                    billing_details: {
                        name: cardHolderName.value,
                    }
                });

                if (error) {
                    this.isLoading = false;
                    this.isError = true;
                    alert('Errore pagamento');

                    
                } else {
                    var payData = {
                        pay : paymentMethod,
                        userId: 1
                    }
                    axios.post('api/payment', payData).then(response => {
                        if(response.status=== 200) {
                            this.isLoading= false;
                            this.isOk= true;
                        }

                    })
                }
            }
        }
    }
</script>

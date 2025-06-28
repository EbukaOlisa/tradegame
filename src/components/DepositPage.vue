<template>
<div>
  <div class="deposit-container">
    <div class="h1">Deposit Funds</div>
    <form @submit.prevent="handleDeposit">
      <div>
        <label for="amount">Deposit Amount</label>
        <input type="number" v-model="amount" id="amount" required />
      </div>
      <button type="submit">Deposit</button>
    </form>
    <div v-if="error" class="error">{{ error }}</div>
  </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex';
import axios from 'axios';

export default {
  data() {
    return {
      amount: 0,
      error: null,
    };
  },
  computed: {
    ...mapGetters(['userId']), // Access userId from the store
  },
  methods: {
    async handleDeposit() {
  try {
    this.error = null;

    const response = await axios.post(
      `${import.meta.env.VITE_APP_BASE_URL}/api/transactions/deposit`,
      { userId: this.userId, amount: this.amount }
    );

    const { txRef, user } = response.data; // ✅ get txRef directly

    // Launch Flutterwave inline modal
    FlutterwaveCheckout({
      public_key: import.meta.env.VITE_FLW_PUBLIC_KEY,
      tx_ref: txRef, // ✅ use directly
      amount: this.amount,
      currency: "NGN",
      customer: {
        email: user.email,
        name: user.name,
      },
      callback: function (payment) {
        alert("Payment completed. You can check your balance shortly.");
      },
      onclose: function () {
        console.log("Payment modal closed");
      },
      customizations: {
        title: "Tr8Game9ja",
        description: "Deposit into your Tr8Game9ja Wallet",
        logo: "/logos.png", // ✅ make sure this image is in the public folder
      },
    });

  } catch (err) {
    this.error = err.response?.data?.error || "An error occurred.";
  }
}


  }
};
</script>


<style>
.error {
  color: red;
  margin-top: 10px;
}
.h1 {
  color: white;
}

.deposit-container  {
  text-align: center;
  margin: 20px auto;
  max-width: 900px;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background-color: lightyellow;
  }
</style>

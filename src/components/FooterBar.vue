<template>
  <footer>
    <div>
      <img src="../assets/tmdb-logo.svg" />
    </div>
    <div></div>
    <div><button @click="donate">Donate</button></div>
  </footer>
</template>

<script>
import axios from "axios";
import Swal from "sweetalert2";
import { mapState, mapActions } from "pinia";
import { useGlobalStore } from "../stores/globalStore";

export default {
  computed: {
    ...mapState(useGlobalStore, ["baseUrl"]),
  },
  methods: {
    ...mapActions(useGlobalStore, ["errorHandler", "successHandler"]),
    async donate() {
      try {
        const errorHandler = this.errorHandler;
        const access_token = localStorage.access_token;

        const { value: amount } = await Swal.fire({
          title: "Enter amount you want to donate",
          input: "number",
          inputLabel: "Your amount",
          showCancelButton: true,
          icon: "question",
          inputValidator: (value) => {
            if (!value) {
              return "You need to write something!";
            }
          },
        });

        if (!amount) {
          throw { message: "Canceled" };
        }

        const { data } = await axios.post(
          this.baseUrl + "/payment",
          {
            amount,
          },
          {
            headers: {
              access_token,
            },
          }
        );

        window.snap.pay(data.transactionToken, {
          onSuccess() {
            Swal.fire({
              icon: "success",
              title: "Success!!",
              text: "Thanks for your donation",
            });
          },
          onClose() {
            errorHandler("You closed the popup without finishing the payment");
          },
        });
      } catch (error) {
        if (error.response) {
          if (error.response.status === 401)
            return this.errorHandler("Login require");
        }
        this.errorHandler(error.message);
      }
    },
  },
};
</script>

<style>
footer img {
  width: 100px;
}
</style>

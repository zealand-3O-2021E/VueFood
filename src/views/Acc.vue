<template>
  {{ message }}
</template>

<script lang="ts">
import {onMounted, ref} from 'vue';
import {useStore} from "vuex";

export default {
  name: "Acc",
  setup() {
    const message = ref('You are not logged in!');
    const store = useStore();

    onMounted(async () => {
      try {
        const response = await fetch('https://rest-service.azurewebsites.net/api/Auth/user', {
          headers: {'Content-Type': 'application/json'},
          credentials: 'include'
        });
        if(response.status != 200) throw new Error()
        const content = await response.json();

        message.value = `Hi ${content.name}`;

        await store.dispatch('setAuth', true);
      } catch (e) {
        await store.dispatch('setAuth', false);
      }
    });

    return {
      message
    }
  }
}
</script>

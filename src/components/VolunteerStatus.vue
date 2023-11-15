<script setup>
import { reactive } from "vue";
import { useVuelidate } from "@vuelidate/core";
import { email, required } from "@vuelidate/validators";

const initialState = {
  bdate: "",
  email: "",
  volunteerStatus: "",
  zip: "",
  msg: "",
  showProgress: false,
  showMsg: false
};

const state = reactive({
  ...initialState,
});

const rules = {
  bdate: { required },
  zip: { required },
  email: { required, email },
};


const v$ = useVuelidate(rules, state);

function clear() {
  v$.value.$reset();

  for (const [key, value] of Object.entries(initialState)) {
    state[key] = value;
  }
}
function submitForm() {
  
  v$.value.$validate();
  if (!v$.value.$error) {
    state.showProgress=true
    getData()
  } 
}
async function getData(){

  const query = `
      {
        updates {
          items {
            email,
            bdate,
            zip,
            volunteerStatus
          }
        }
      }`;
      
  const endpoint = "/data-api/graphql";

  try{
  const response = await fetch(endpoint, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ query: query })
  })
  const result = await response.json();
  processNewData(result);
  } catch (err) {
    // console.log(err)
    state.showProgress=false
    state.showMsg=true
    state.msg="Your info found no match"
  }
}

function processNewData(items){
  const volunteerData = items.data.updates.items

  const filteredItem = volunteerData.filter(item => item.email === state.email && item.zip === state.zip && item.bdate === state.bdate)
  // console.log(filteredItem)
  state.showMsg = true

  // console.log(filteredItem.length)
  // console.log(filteredItem)
  // console.log(filteredItem[0].volunteerStatus)

  state.volunteerStatus = filteredItem[0].volunteerStatus

  if(filteredItem.length < 1){
    state.msg = "Your info found no match..."
  } else if(filteredItem.length > 0 && !state.volunteerStatus) {
    state.msg = "There's no update yet..."
  } else {
    state.msg = "Your update is: " + state.volunteerStatus
  }
  
  state.showProgress = false

}
</script>
<template>
  <v-overlay v-model="state.showProgress" class="justify-center align-center">
    <v-progress-circular
      indeterminate
      size="60"
      width="5"
      color="primary"
    ></v-progress-circular>
  </v-overlay>

  <v-sheet width="300" class="mx-auto mt-15">
    <form>
      <v-text-field
        v-model="state.email"
        :error-messages="v$.email.$errors.map((e) => e.$message)"
        label="Enter Email"
        variant="outlined"
        required
        @input="v$.email.$touch"
        @blur="v$.email.$touch"
      ></v-text-field>

      <v-text-field
        v-model="state.bdate"
        :error-messages="v$.bdate.$errors.map((e) => e.$message)"
        label="Enter Birthdate"
        variant="outlined"
        required
        @input="v$.bdate.$touch"
        @blur="v$.bdate.$touch"
      ></v-text-field>

      <v-text-field
        v-model="state.zip"
        :error-messages="v$.zip.$errors.map((e) => e.$message)"
        label="Enter Zip"
        variant="outlined"
        required
        @input="v$.zip.$touch"
        @blur="v$.zip.$touch"
      ></v-text-field>

      <v-btn class="me-4 mt-5" color="primary" @click="submitForm"> submit </v-btn>
      <v-btn class="mt-5" @click="clear"> clear </v-btn>

      <v-textarea class="mt-15" v-if="state.showMsg" v-model="state.msg" variant="outlined"></v-textarea>
    </form>
  </v-sheet>
</template>

<template>
  <v-dialog v-model="dialog" hide-overlay transition="dialog-bottom-transition" persistent>
    <v-card>

      <v-card-title>Edit Exercise</v-card-title>

      <v-card-text>
        <v-form v-model="validForm">
          <v-text-field label="Name" :value="data.name" @input="exercise.name=$event"
            :rules="rules.name" outlined/>

          <v-textarea label="Detail" :value="data.detail" @input="exercise.detail=$event"
            :rules="rules.detail" outlined/>

          <v-text-field label="Duration" :value="data.duration" @input="exercise.duration=$event"
            :rules="rules.duration" type="numeric" outlined/>

          <v-text-field label="Repetitions" :value="data.repetitions" @input="exercise.repetitions=$event"
            :rules="rules.repetitions" type="numeric" outlined/>

          <v-select label="Type" :items="types" :value="data.type" @input="exercise.type=$event"
            :rules="rules.type" item-text="name" item-value="value" outlined/>
        </v-form>
      </v-card-text>

      <v-card-actions>
        <v-btn color="grey lighten-1" @click="$emit('cancel');" bottom large>Cancel</v-btn>
        <v-spacer />
        <v-btn :disabled="!validForm" color="primary text-center black--text" class="accent" @click="save()" bottom large>Save</v-btn>
      </v-card-actions>

      <v-overlay :value="showOverlay">
        <v-progress-circular indeterminate size="64"/>
      </v-overlay>

      <v-snackbar v-model="showSnackbar">
          {{ snackbarText }}
      </v-snackbar>
      
    </v-card>
  </v-dialog>
</template>


<script>
  import Vue from 'vue';
  import { RoutineApi } from '../../api/routines';

  export default {
      props: ['dialog', 'data'],

      methods: {
        async save(){
          try{
            this.showOverlay = true;

            let routineId = this.exercise.routineId;
            delete this.exercise.routineId;

            let cycleId = this.exercise.cycleId;
            delete this.exercise.cycleId;

            this.exercise.duration=parseInt( this.exercise.duration);
            this.exercise.repetitions=parseInt( this.exercise.repetitions);
            
            await RoutineApi.updateExercise(routineId, cycleId, this.exercise.id, this.exercise);

            this.exercise.routineId = routineId;
            this.exercise.cycleId = cycleId;

            this.showOverlay = false;
            this.$emit('save', this.exercise);
          }catch(e){
              this.snackbarText = 'Ups! Something went wrong';
              console.log(e);

              this.showOverlay = false;
              this.showSnackbar = true;
          }
        }
      },

      data: function(){
        return {
          showOverlay: false,
          showSnackbar: false,
          snackbarText: '',

          validForm: false,

          types: [{name: 'Exercise', value: 'exercise'}, {name: 'Rest', value: 'rest'}],

          exercise: Vue.util.extend({}, this.data),

          rules: {
            name: [ v => !!v || 'Name is required',
              v => (v && v.length < 100) || 'Name must be less than 100 characters'],
            detail: [ v => !!v || 'Detail is required',
              v => (v && v.length < 200) || 'Detail must be less than 200 characters'],
            duration: [ v => !!v || 'Duration is required',
              v => v > 0 || 'Duration must be positive'],
            repetitions: [ v => !!v || 'Repetitions is required', v => v > 0
              || 'Repetitions must be positive'],
            type: [ v => !!v || 'Type is required'],
            cycle: [ v => !!v || 'Cycle is required'],
          }
        }
      }

  }
</script>
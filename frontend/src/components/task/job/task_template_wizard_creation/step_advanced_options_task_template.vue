<template>
  <v-container fluid data-cy="task-template-advanced-options-step" v-if="job">
    <div class="d-flex mb-8 justify-space-between">
      <h1 class="font-weight-medium text--primary mr-4" data-cy="task-template-advanced-options-step-title">
        Advanced Options:
      </h1>
    </div>

    <v_error_multiple :error="error"></v_error_multiple>
    <p class="text--primary">
      Optional Configurations.
    </p>


    <v-container fluid>

      <userscript_select
        data-cy="userscrips-select"
        :project_string_id="project_string_id"
        @change="job.default_userscript_id = $event.id"
        label="Choose a Default Userscript"
      >
      </userscript_select>

      <diffgram_select
        :item_list="file_handling_list"
        data-cy="file-handling-select"
        v-model="job.file_handling"
        label="File Handling"
        :disabled="loading"
      >
      </diffgram_select>

    </v-container>

    <wizard_navigation
      @next="on_next_button_click"
      :next_visible="true"
      :loading_next="loading_steps"
      :disabled_next="loading_steps"
      @back="$emit('previous_step')"
      :skip_visible="true">

    </wizard_navigation>
  </v-container>

</template>

<script lang="ts">

  import axios from 'axios';
  import guide_selector from '../../guide/guide_selector'
  import userscript_select from '../../../annotation/userscript/userscript_select'

  import Vue from "vue";

  export default Vue.extend({
      name: 'step_guides_task_template',
      props: [
        'project_string_id',
        'job',
        'loading_steps',
      ],

      components: {
        guide_selector,
        userscript_select,
      },

      data() {
        return {
          error: {},
          show_credentials: false,
          loading: false,
          type_list: [
            {
            display_name: 'Normal',
            value: 'Normal'
            },
            {
              display_name: 'Exam',
              value: 'exam_template'
            }
          ],
          share_list: [],
          file_handling_list: [
            {
              'display_name': 'Use Existing (Default)',
              'name': 'use_existing',
              'icon': 'mdi-cached',
              'color': 'primary'
            },
            {
              'display_name': 'Isolate (New Versions of Files)',
              'name': 'isolate',
              'icon': 'mdi-ab-testing',
              'color': 'primary'
            }
          ],
        }
      },
      created() {

      },

      methods: {
        on_next_button_click: function () {
          this.$emit('next_step');
        }
      }
    }
  ) </script>

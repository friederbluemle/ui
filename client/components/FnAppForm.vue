<template>
  <modal :title="title()" :show="show" @closed="closed" @ok="ok" @cancel="closed">
    <form class="form-horizontal" v-on:submit.prevent="ok">
      <div class="form-group">
        <label class="col-sm-3 control-label">Name</label>
        <div class="col-sm-9">
          <input type="text" class="form-control" placeholder="e.g. my-app" v-model="app.name" required :disabled="edit" @keydown.enter.prevent="">
        </div>
      </div>

      <hr>

      <div class="form-group">
        <label class="col-sm-3 control-label">Config</label>
        <div class="col-sm-9">
          <div class="row" v-for="(line, index) in appConfig">
            <div class="col-sm-5 cfg-key">
              <input type="text" class="form-control" placeholder="Key" v-model="line.key" @keydown.enter.prevent="">
            </div>
            <div class="col-sm-5 cfg-val">
              <input type="text" class="form-control" placeholder="Value" v-model="line.value" @keydown.enter.prevent="">
            </div>
            <div class="col-sm-1 toolbar">
              <button class="btn btn-default" @click.prevent="removeConfigLine(index)"><i class="fa fa-times"></i></button>
            </div>
          </div>
          <div>
            <a href="#" class="" @click.prevent="addConfigLine">
              <i class="fa fa-plus"></i> Add line
            </a>
          </div>
        </div>
      </div>

    </form>

    <div slot="footer">
      <button type="button" class="btn btn-primary" @click="ok">{{okBtnName()}}</button>
    </div>
  </modal>
</template>

<script>
import Modal from '../lib/VueBootstrapModal.vue';
import { eventBus } from '../client';
import { defaultErrorHandler, configToLines, linesToConfig, getAuthToken } from '../lib/helpers';

export default {
  props: [],
  components: {
    Modal
  },
  data: function(){
    return {
      show: false,
      edit: false,
      app: {},
      appConfig: [{key: "", value: ""}]
    }
  },
  methods: {
    title: function(){
      return this.edit ? 'Edit App' : 'Create New App';
    },
    okBtnName: function(){
      return this.edit ? 'Save Changes' : 'Create App';
    },
    closed: function(){
      this.show = false;
    },
    addConfigLine: function(){
      this.appConfig.push({key: "", value: ""});
    },
    removeConfigLine: function(index){
      this.appConfig.splice(index, 1)
    },
    ok: function(){
      var t = this;
      eventBus.$emit('NotificationClear');
      this.app.config = linesToConfig(this.appConfig);
      if (this.edit) {
        var url = '/api/apps/' + encodeURIComponent(this.app.name);
      } else {
        var url = '/api/apps';
      };
      $.ajax({
        headers: {'Authorization': getAuthToken()},
        url: url,
        method: this.edit ? 'PATCH' : 'POST',
        data: JSON.stringify(this.app),
        contentType: "application/json",
        dataType: 'json',
        success: (res) => {
          if (this.edit){
            eventBus.$emit('AppUpdated', res.app);
          } else {
            eventBus.$emit('AppAdded', res.app);
          }
          t.closed();
          t.app = {};
        },
        error: defaultErrorHandler
      })
    },
  },
  created: function (){
    eventBus.$on('openEditApp', (app) => {
      this.app = jQuery.extend(true, {}, app);
      this.appConfig = configToLines(app.config)
      this.edit = true;
      this.show = true;
    });
    eventBus.$on('openAddApp', () => {
      this.app = {};
      this.appConfig = configToLines({});
      this.edit = false;
      this.show = true;
    });
  }
}
</script>

<style scoped>
.cfg-key {
  padding: 0 5px 5px 15px;
}
.cfg-val {
  padding: 0 5px 5px 5px;
  margin-right: -20px;
  width: 50%;
}
</style>

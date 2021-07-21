<template>
  <div v-click-outside="close" class="b-overlay" :class="iternalIsOpen ? 'b-overlay-isOpen' : ''">
      <b-form class="b-overlay-cta" :fields="fields">
        <b-btn slot="submit" submit @click="submit">Absenden!</b-btn>
      </b-form>
  </div>
</template>

<script>
import BBtn from "@/components/base/btn/BBtn";
import BForm from "@/components/base/form/BForm";
import axios from "axios";
import ClickOutside from 'vue-click-outside'
import {validator} from "@/assets/content";
export default {
  name: "BOverlay",
  components: {BForm, BBtn},
  data: () => ({
    iternalIsOpen: false,
    fields: []
  }),
  directives: {
    ClickOutside
  },
  mounted () {
    // prevent click outside event with popupItem.
    this.popupItem = this.$el
    import('@/assets/content').then(v => this.fields = v.getForm())
  },
  props: {
    value: Boolean
  },
  watch: {
    value: {
      immediate: true,
      handler: function (val) {
        this.iternalIsOpen = val
      }
    }
  },
  methods: {
    close() {
      this.iternalIsOpen = false
      this.$emit('input', false)
    },
    submit() {
      const body = Object.assign({},this.arrayToObject(this.fields, "id"));
      if (this.sendMail(body)) {
        this.close()
      } else {
        this.$forceUpdate()
      }
    },
    arrayToObject(arr, key) {
      return arr.reduce((acc, e) => {
        const newKey = e[key];
        delete e[key]
        Object.assign(acc, {[newKey]: e})
        return acc;
      }, {})
    },
    sendMail(body) {
      const url = "https://f1rf-backend.herokuapp.com";
      const isValidationError = this.fields
          .map(f => validator(f))
          .some(v => {
            return !!v?.validationMessage
          });
      if (!isValidationError) {
        axios.create({
          baseURL: url,
          headers: {
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Methods": "GET, POST, PATCH, PUT, DELETE, OPTIONS",
            "Access-Control-Allow-Headers": "Origin, Content-Type, X-Auth-Token"
          }})
            .post("/f1rf/api/mails", body)
            .then(() => {})
            .catch(() => {})
        return true;
      } else {
        return false;
      }
    }
  }
}
</script>

<style scoped>
.b-overlay {
  position: absolute;
  height: 76vh;
  width: 100vw;
  bottom: 0;
  left: 0;
  transform: translateY(90vh);
  transition: 250ms ease-in transform;
  opacity: 0;

  padding: 48px 48px 48px 48px;
  border-top-right-radius: 32px;
  border-top-left-radius: 32px;
  box-shadow: 0px -10px 20px rgba(0,0,0, 0.2);
  background-color: rgb(50,50,50);


}

.b-overlay .b-overlay-cta {
  /*justify-self: flex-end;*/
  display: grid;
  grid-template-columns: 100%;
  grid-row-gap: 32px;
}

.b-overlay-isOpen {
  opacity: 1;
  transform: translateY(0vh);
  transition: 0.3s ease-out transform;
}
</style>

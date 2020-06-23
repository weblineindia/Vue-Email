<template>
  <div class="list-view">
    <div>
      <input
        :id="id + index"
        v-model="valueData[index].email"
        v-validate="{
          emailValidate,
          isUniqueEmail
        }"
        :disabled="disabled"
        type="text"
        :placeholder="placeholder"
        :tabindex="tabindex"
        autocomplete="off"
        :maxlength="maxlength"
        :name="name + index"
        :class="[
          {
            input: true,
            'form-control': true,
            'text-visible': valueData[index].email ? true : false,
            'is-danger': errors.has(name + index),
            'error-text': errors.has(name + index),
            'mt-4':
              isFocusField || valueData[index].email !== '' || isRequiredFiled || isFirstField
                ? false
                : true,
          },
        ]"
        @keypress.enter.prevent
        @focus="onFocus($event, index)"
        @input="onChangeField($event)"
        @blur="onBlur($event)"
      />
      <span v-if="isMultiple" v-show="valueData[index].email !== ''" class="add-remove">
        <span>
          <font-awesome-icon icon="minus" />
        </span>
        <span v-show="isShowPlus">
          <font-awesome-icon
            icon="plus"
            class="pointer"
            @click.prevent="onMultipleEmail(id, index)"
          />
        </span>
      </span>
    </div>
    <div v-if="errors.items.length > 0">
      <div v-for="(error, key) in errors.items" :key="key">
        <p
          v-if="errors.has(name + index)"
          :class="{
            control: true,
            'error-msg': errors.has(name + index),
          }"
        >
          <span
            v-show="
                  (
                    error.rule === 'emailValidate') &&
                    error.field === name + index
                "
          >{{emailFormateError}}</span>
          <span
            v-show="(error.rule === 'isUniqueEmail') && error.field === name + index"
          >{{duplicateEmailError}}</span>
        </p>
      </div>
    </div>
  </div>
</template>
<script>
// For load Regex

import * as Regex from "./regex.js";
import Vuelidate from "vuelidate";
import Vue from "vue";
import * as VeeValidate from "vee-validate";
// Load font awesome
import { library } from "@fortawesome/fontawesome-svg-core";
import {
  faPlus,
  faMinus,
  faCircle,
  faClone
} from "@fortawesome/free-solid-svg-icons";
import "@fortawesome/fontawesome-svg-core/styles.css";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";
library.add(faPlus, faMinus, faCircle, faClone);

Vue.component("font-awesome-icon", FontAwesomeIcon);

Vue.use(Vuelidate);
Vue.use(VeeValidate, {
  events: "change|blur"
});

// const keepEmailTemp = []

export default {
  data() {
    return {
      isShowErrorInModalForUniqueness: false,
      errorContentForUniqueness: "",
      isRequiredFiled: false,
      value: [{ email: "" }],
    };
  },
  props: {
    maxlength: {
      type: Number,
      default: 50
    },
    id: {
      type: String,
      default: ""
    },
    emailFormateError: {
      type: String,
      default: "Email is not valid"
    },
    duplicateEmailError: {
      type: String,
      default: "Do not enter same email"
    },
    onMultipleEmail: {
      type: Function,
      default: () => {}
    },
    name: {
      type: String,
      default: "email"
    },
    index: {
      type: Number,
      default: 0
    },
    values: {
      type: Array[Object],
      default: () => [
        {
          email: ""
        }
      ]
    },
    isMultiple: {
      type: Boolean,
      default: true
    },
    isShowPlus: {
      type: Boolean,
      default: false
    },
    errorShowModalEmail: {
      type: Boolean,
      default: false
    },
    placeholder: {
      type: String,
      default: "Email"
    },
    disabled: {
      type: Boolean,
      default: false
    },
    tabindex: {
      type: Number,
      default: 0
    }
  },
  computed: {
    valueData: {
      get() {
        return this.values;
      },
      set(newData) {
        return (this.valueData = newData);
      }
    }
  },
  created() {
    /**
     * email formate validation
     */
    this.$validator.extend("emailValidate", {
      getMessage: () => "uiyhuiy",
      validate: value => {
        return new Promise(resolve => {
          const validEmailRegex = Regex.EMAILFORMATE_REGEX;
          if (!validEmailRegex.test(value)) {
            resolve({ isVerifiedEmail: false });
          } else {
            resolve({ isVerifiedEmail: true });
          }
        })
          .catch(function(error) {
            console.log(error);
          })
          .then(function(data) {
            if (data) {
              return data.isVerifiedEmail;
            } else {
              return true;
            }
          });
      }
    });

    /**
     * for uniqueness check in email
     */
    this.$validator.extend("isUniqueEmail", {
      getMessage: () => "jkhkjh",
      validate: value => {
        return new Promise(resolve => {
          console.log("fsfsdf*******", this.values, event);
          if (
            event !== undefined &&
            event.target !== undefined &&
            event.target.name !== "" &&
            event.target.name !== undefined
          ) {
            const res = event.target.name.split("_");
            console.log("res", res);
            if (res[1] !== isNaN && res[1] !== undefined) {
              if (value) {
                if (this.values.length > 1) {
                  const tempArray = [];
                  for (let i = 0; i <= this.values.length - 1; i++) {
                    if (parseInt(i) !== parseInt(res[1])) {
                      if (
                        this.values[i].email ===
                        this.values[parseInt(res[1])].email
                      ) {
                        tempArray.push(this.values[i]);
                      }
                    }
                  }
                  if (tempArray.length > 0) {
                    resolve({ isUniqeEmailValidation: false });
                  } else {
                    resolve({ isUniqeEmailValidation: true });
                  }
                } else {
                  resolve({ isUniqeEmailValidation: true });
                }
              }
            }
          }
        })
          .catch(function(error) {
            console.log(error);
          })
          .then(function(data) {
            if (data) {
              return data.isUniqeEmailValidation;
            } else {
              return true;
            }
          });
      }
    });

  },
  methods: {
    onFocus(event, index) {
      this.$emit("focus", event, index);
    },
    onChangeField() {
      this.valueData[this.index].email = this.valueData[
        this.index
      ].email.replace(Regex.EMAIL_NOTALLOW_SPECIALC_REGEX, "");
      this.valueData[this.index].email = this.valueData[
        this.index
      ].email.toLowerCase();
      this.$emit("onChange", this.valueData, this.id, this.index, this.errors);
    },
    onBlur() {
      this.$emit("onBlur", event, this.placeholder);
    }
  
  }
};
</script>
<style  scoped>
@import './style.css';
</style>
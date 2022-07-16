
<script>
  import Inputmask from 'inputmask';
  import { defineComponent, reactive, nextTick, watch, ref, onUnmounted } from "vue";

export default defineComponent({
  props: ['value'],
  emits : ['input'],
  setup(props, { emit }) {

    let message=reactive('');
    let input = reactive();
    let invalid = ref(false);
    let result = reactive(props.value || '20002200');

    const setInput = (el) => {
      if (!el)
        return;

      if (!input) {
          input=new Inputmask('(Hh:Mm - Hh:Mm \\/ ){1,3}|(U)|(S)', { 
          groupSeparator: "\\/ ",
          jitMasking : true,
          clearMaskOnLostFocus: false, 
          clearIncomplete: false,
          autoUnmask : true,
          tabThrough: false,
          nullable: true,
          oncomplete: () => {
            result = input.unmaskedvalue();
            input.setValue(result);
            emit('input', `${result}`);
          },
          onKeyValidation: () => {
            invalid = isInvalid();
            el.style.borderColor = invalid ? 'red' : 'green'
            result = input.unmaskedvalue();
            console.log('input', invalid ? null : result);
            emit('input', invalid ? null : `${result}`);
          },
          definitions : {
            'H' : { validator : "[0-2]" },
            'h' : { validator : "[0-9]"},
            'M' : { validator : "[0-5]" },
            'm' : {validator : "[0-9]" },
            'U' : { validator : "O" },
            'S' : { validator : "I" }
          }
        }).mask(el);
        input.setValue(result);
      }
    };

    const isInvalid = () => {
      let orderedTimes;
      const val = input.unmaskedvalue()
      const length = val.length;
      const vals = val.match(/.{1,4}/g)
      if (vals && vals.length < 2)
        orderedTimes = true;
      else if (Array.isArray(vals) && vals.every(v => v.length === 4)) { // On découpe tout ce petit monde en 4 caractères

        const pseudoSortedTimes = ([...vals].sort((a,b) => {
          if (parseInt(a) < 400) // On considère que les heures du jour vont jusqu'à 4 du matin
            a = `${2000}+${parseInt(a)}`
          return a-b
        }));

        orderedTimes = pseudoSortedTimes + ' ' === vals + ' '; // On vérifie que les heures sont bien ordonnées en sortant et comparant à la valeur originale

        if (!orderedTimes) {
          message = 'Les horaires doivent être ordonnés';
        } else {
          message = null;
        }
      }
      return !(orderedTimes && input.isValid() && ( (length % 8) ===0  || (length <= 1)));
    } 

    onUnmounted(() => {
      if ('destroy' in input)
        input.destroy();
      input = null;
      result = null
    });

    return {
      setInput,
      result,
      message,
    }
  }
});
</script>


<template>
    <div class="redval-input">
        <input class="input-perse"
          :ref="setInput"
          name="horaire"
        />
        <span v-if="invalid" class="redval-message">{{message}}</span>
    </div>
</template>

<style>
.input-perse {
  width: 100%!important;
  border: 2px solid;
  border-color : #d3dae4;
  border-radius: 6px;
  padding : 16px!important
}

.redval-message {
  color : red;
}



</style>


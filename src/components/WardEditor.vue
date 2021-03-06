<template>
  <div class="w-row ward-card">
    <div class="w-col">
      <h4 style="text-align:center; margin-bottom: 10px;">
        {{ wardToEditId === 0 ? 'Add' : 'Update' }} Ward Information
      </h4>
    </div>
    <div class="w-col">
      <a v-if="wardToEditId !== 0"
        href="#"
        class="button-3 w-button"
        style="float: right; margin-right: 10px;"
        v-on:click="removeWard">
        - REMOVE WARD
      </a>
    </div>
    <form v-on:submit.prevent="submitChanges">
      <div class="w-row">
      <div
        v-for="(wardField, index) in wardForm"
        class="w-col w-col-6"
        :key="index">
        <div class="form-block-3 w-form">
          <div class="subheading" style="border: 0px;">
            <label class="field-label">
              <strong>{{ wardField[0] }}</strong>
            </label>
            <template
              v-if="wardField[2] == 'text' || wardField[2] == 'number'">
              <input
                :type="wardField[2]" class="ward-text-field w-input"
                v-model="wardModelFields[index]" required
                min="0"
                maxlength="256"
              />
            </template>
            <template v-else-if="wardField[2] == 'option'" >
              <select class="ward-field-design w-input"
                v-model="wardModelFields[index]">
                <option
                  v-for="(val, index) in wardField[3]"
                  :value="val[0]" :key="index">
                  {{ val[1] }}
                </option>
              </select>
            </template>
          </div>
        </div>
      </div>
      </div>
      <div class="w-row" style="margin-top: 20px;">
        <div class="form-fail" v-show="error != ''">
          <div><strong>{{ error }}</strong></div>
        </div>
        <div class="w-col w-col-6">
          <input
            type="button" class="signup-button status-form clear-form w-button"
            value="GO BACK" v-on:click="$emit('edit-done', 0);">
        </div>
        <div class="w-col w-col-6">
          <input
            type="submit" class="signup-button status-form profile-pahe w-button"
            value="Save changes"
          >
        </div>
      </div>
    </form>
  </div>
</template>


<script>
import Vue from 'vue';
import Component from 'vue-class-component';
import API from '../utils/apis';
import Utils from '../utils/utils';


function getOptionValueList(options) {
  const optionValueList = [];
  for (let i = 0; i < options.length; i += 1) {
    optionValueList.push([options[i], options[i]]);
  }
  return optionValueList;
}

const yesNoOptions = [
  ['true', 'Yes'],
  ['false', 'No'],
];


function getDefaultWard() {
  return ({
    name: '',
    id: 0,
    buildingName: '',
    floor: '',
    totalBeds: 0,
    gender: '',
    covidWard: 'true',
    covidStatus: '',
    severity: '',
    ventilators: 0,
    ventilatorsOccupied: 0,
    extraFields: {},
  });
}


const WardEditorProps = Vue.extend({
  props: {
    wardToEdit: {
      required: false,
    },
    wardToEditId: {
      required: true,
      type: Number,
    },
  },
});

@Component
export default class WardEditor extends WardEditorProps {
  wardForm = [
    ['Building Name/Number', 'buildingName', 'text'],
    ['Floor', 'floor', 'option', getOptionValueList(
      ['LG', 'G', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10'],
    )],
    ['Ward Name/Number', 'name', 'text'],
    ['Total beds', 'totalBeds', 'number'],
    ['Gender', 'gender', 'option', getOptionValueList(['MALE', 'FEMALE', 'UNISEX'])],
    ['Is it COVID Ward?', 'covidWard', 'option', yesNoOptions],
    [
      'Patient COVID status',
      'covidStatus',
      'option',
      getOptionValueList(['CONFIRMED', 'SUSPECTED']),
    ],
    ['Patient severity', 'severity', 'option', getOptionValueList(['MILD', 'MODERATE', 'SEVERE'])],
    ['Total ventilators', 'ventilators', 'number'],
    ['Ventilators occupied', 'ventilatorsOccupied', 'number'],
    [
      'Is distance between beds more than 6 feet?',
      'extraFields.minDistanceBetBeds', 'option', yesNoOptions,
    ],
    [
      'Number of independent single room beds with attached washroom',
      'extraFields.independentBeds', 'number',
    ],
    ['Number of oxygen cylinders available', 'extraFields.oxygenCylinder', 'number'],
    [
      'Number of beds with wall oxygen supplies available',
      'extraFields.wallOxygenSuppliedBeds', 'number',
    ],
    [
      'Number of beds with Central Oxygen system',
      'extraFields.centralOxygenSuppliedBeds', 'number',
    ],
    ['Number of pulse oximeters available', 'extraFields.pulseOximeters', 'number'],
    ['Number of infusion pumps available', 'extraFields.infusionPumps', 'number'],
  ];

  error = '';

  submitChanges() {
    this.ward.wardId = this.wardToEditId;
    this.updateWardWithModelFields();
    API.saveWard(this.$store.state.user.facilityId, this.ward).then(
      () => {
        const action = this.wardToEditId === 0 ? 'added' : 'updated';
        alert(`Ward details ${action}`); // eslint-disable-line
        this.$emit('edit-done', 1);
      }, () => {
        // console.log(error);
        this.error = 'Error: (building name, floor and ward name) should be unique.';
      },
    );
  }

  removeWard() {
    const confirmRes = confirm('Are you sure you want to remove this ward?'); // eslint-disable-line
    if (confirmRes) {
      API.removeWard(this.$store.state.user.facilityId, this.wardToEditId).then(
        (success) => {
          if (success.error) {
            Utils.standardErrorHandler(success.error);
            alert(`Error: ${success.error.errorMsg}`);
          } else {
            alert('Ward successfully removed');
            this.$emit('edit-done', 1);
          }
        }, () => {
          alert(
            'Error in removal, please try again. '
            + 'If problem persists then contact system administrator.',
          );
        },
      );
    }
  }

  updateWardWithModelFields() {
    for (let i = 0; i < this.wardForm.length; i += 1) {
      // wardForm.push(this.wardForm[i]);
      if (this.wardForm[i][1].startsWith('extraFields')) {
        this.ward.extraFields[this.wardForm[i][1].split('.')[1]] = this.wardModelFields[i];
      } else {
        this.ward[this.wardForm[i][1]] = this.wardModelFields[i];
      }
    }
  }

  get wardModelFields() {
    const wardModel = [];
    if (!this.ward.extraFields) {
      this.ward.extraFields = {};
    }
    for (let i = 0; i < this.wardForm.length; i += 1) {
      // wardForm.push(this.wardForm[i]);
      if (this.wardForm[i][1].startsWith('extraFields')) {
        wardModel.push(this.ward.extraFields[this.wardForm[i][1].split('.')[1]]);
      } else {
        wardModel.push(this.ward[this.wardForm[i][1]]);
      }
    }
    return wardModel;
  }

  get ward() {
    if (!this.wardToEdit) {
      return getDefaultWard();
    }
    return this.wardToEdit;
  }
}
</script>

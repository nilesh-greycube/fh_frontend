<template>
  <div>
    <div class="p-4">
      <div class="flex items-center bg-[#265166] rounded h-12 p-4">
        <p class="font-bold text-white">Enter Grade Details</p>
      </div>

      <div class="flex justify-between p-4">
        <div class="w-[30%] space-y-2">
          <p>City you are seeking admission in?</p>
          <Select v-model="admission_city" class="w-full" :options="city_options"></Select>
        </div>

        <div class="w-[30%] space-y-2">
          <p>Child's DOB</p>
          <DatePicker v-model="child_dob" placeholder="Select Date" label="Label" />
        </div>

        <div class="w-[30%] space-y-2">
          <p>Which Acadamic Year are you applying for?</p>
          <Select v-model="academic_year" class="w-full" :options="academic_year_options"></Select>
        </div>
      </div>

      <Button class="m-4" variant="solid" @click="calculate_grade">Calculate Grade</Button>
    </div>

    <hr>

    <div class="p-4">
      <div class="p-4 prose prose-table" v-html="grades_table"></div>

      <div class="flex justify-between p-4 w-[50%]">
        <div class="space-y-2 w-[49%]">
          <p>Select Grade</p>
          <Select class="w-full" v-model="selected_grade" :options="unique_grades"></Select>
        </div>

        <div class="space-y-2 w-[49%]">
          <p>Select School</p>
          <Select class="w-full" v-model="selected_school" :options="unique_schools"></Select>
        </div>
      </div>
    </div>

    <hr>
  </div>
</template>
<script setup>
import { Button, FormControl, FormLabel, Select, DatePicker, createResource, useAxisChartOptions } from "frappe-ui"
import { ref, watch } from 'vue';

const child_dob = ref(null)
const admission_city = ref(null)
const academic_year = ref(null)

const eligible_grades = ref(null)
const grades_table = ref(null)
const unique_grades = ref(null)
const unique_schools = ref(null)

const selected_grade = ref(null)
const selected_school = ref(null)

let city_options = [
  {
    label: "Surat",
    value: "Surat"
  },
  {
    label: "Vapi",
    value: "Vapi"
  },
  {
    label: "Aurangabad (CSN)",
    value: "Aurangabad (CSN)"
  },
]

let academic_year_options = [
  {
    label: "2025-26",
    value: "2025-26"
  },
  {
    label: "2026-27",
    value: "2026-27"
  },
  {
    label: "2027-28",
    value: "2027-28"
  },
  {
    label: "2028-29",
    value: "2028-29"
  },
  {
    label: "2029-30",
    value: "2029-302025-26"
  },
]

watch([child_dob, admission_city, academic_year], () => {
  calculate_grade(child_dob, admission_city, academic_year)
})

watch(selected_grade, () => {
  get_unique_schools_based_on_grade(JSON.stringify(eligible_grades.value), selected_grade.value)
})

function calculate_grade() {
  if (admission_city.value && child_dob.value && academic_year.value) {
    const call_calculator = createResource({
      url: "fh_admission.api.get_eligible_grades",
      params: {
        child_dob: child_dob.value,
        child_academic_year: academic_year.value,
        city: admission_city.value
      },
      auto: true,
      onSuccess: (data) => {
        selected_grade.value = null
        selected_school.value = null

        eligible_grades.value = data
        get_html_tables(data)
        get_unique_grades(JSON.stringify(data))
      }
    })
  }
}

function get_html_tables(data) {
  const call_html_table_generator = createResource({
    url: "fh_admission.api.generate_eligibility_html_tables",
    params: {
      data: data
    },
    auto: true,
    onSuccess: (data) => {
      grades_table.value = data
    }
  })
}

function get_unique_grades(data) {
  const call_html_table_generator = createResource({
    url: "fh_admission.api.get_unique_grades",
    params: {
      query_results: data
    },
    auto: true,
    onSuccess: (data) => {
      unique_grades.value = data
    }
  })
}

function get_unique_schools_based_on_grade(data, grade) {
  const call_html_table_generator = createResource({
    url: "fh_admission.api.get_unique_schools_based_on_grade",
    params: {
      query_results: data,
      grade: grade
    },
    auto: true,
    onSuccess: (data) => {
      unique_schools.value = data
    }
  })
}
</script>

<style scoped>
table,
tr,
th {
  border: 1px solid black;
}
</style>
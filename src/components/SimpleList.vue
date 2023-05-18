<script lang="ts" setup>
import email_names from "@/utils/email_names.json";
import email_numbers from "@/utils/email_numbers.json";
import { onMounted, ref } from "vue";

interface EmailNames {
  id: number;
  first_name: string;
  last_name: string;
  email: string;
}
interface EmailNumbers {
  id: number;
  email: string;
  cc_number: string;
}
interface CombinedNamesNumbers {
  id?: number;
  email?: string;
  cc_number?: string;
  first_name?: string;
  last_name?: string;
}

const emailNames: EmailNames[] = email_names;
const emailNumbers: EmailNumbers[] = email_numbers;

const filterData = async (): Promise<CombinedNamesNumbers[]> => {
  const onlyEmailFromEmailNumbers = await emailNumbers.map(
    (item) => item.email
  );
  const includeEmail = await emailNames.filter((item) =>
    onlyEmailFromEmailNumbers.includes(item.email)
  );

  const email = await includeEmail.map((item) => item.email);

  const toFindDuplicates = (email: string[]) =>
    email.filter(
      (item: string, index: number) => email.indexOf(item) !== index
    );
  const duplicateElements = toFindDuplicates(email);
  const uniq = email.filter((item) => !duplicateElements.includes(item));

  const noDuplicateEmailNames = await emailNames.filter((item) =>
    uniq.includes(item.email)
  );
  const noDuplicateEmailNumber = await emailNumbers.filter((item) =>
    uniq.includes(item.email)
  );

  let merged = [];

  for (let i = 0; i < noDuplicateEmailNames.length; i++) {
    merged.push({
      ...noDuplicateEmailNames[i],
      ...noDuplicateEmailNumber.find(
        (itmInner) => itmInner.email === noDuplicateEmailNames[i].email
      ),
    });
  }

  return merged;
};

const filteredData = ref<CombinedNamesNumbers[]>([]);

onMounted(async () => {
  filteredData.value = await filterData();
});
</script>

<template>
  <div class="w-full flex gap-2 flex-col text-gray-50">
    <div
      v-for="(item, i) in filteredData"
      :key="i"
      class="flex flex-col bg-blue-400 p-4 border border-none rounded-md gap-2"
    >
      <div class="font-bold uppercase text-xl">
        {{ item.first_name + " " + item.last_name }}
      </div>
      <div class="flex gap-2">
        <div class="flex gap-1">
          <span class="font-bold text-gray-100">Email:</span>
          <span>{{ item.email }}</span>
        </div>
        <div class="flex gap-1">
          <span class="font-bold text-gray-100">CC Number:</span>
          <span>{{ item.cc_number }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

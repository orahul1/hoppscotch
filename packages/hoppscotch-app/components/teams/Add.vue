<template>
  <SmartModal v-if="show" :title="t('team.new')" @close="hideModal">
    <template #body>
      <div class="flex flex-col px-2">
        <input
          id="selectLabelTeamAdd"
          v-model="name"
          v-focus
          class="input floating-input"
          placeholder=" "
          type="text"
          autocomplete="off"
          @keyup.enter="addNewTeam"
        />
        <label for="selectLabelTeamAdd">
          {{ t("action.label") }}
        </label>
      </div>
    </template>
    <template #footer>
      <span>
        <ButtonPrimary :label="t('action.save')" @click.native="addNewTeam" />
        <ButtonSecondary
          :label="t('action.cancel')"
          @click.native="hideModal"
        />
      </span>
    </template>
  </SmartModal>
</template>

<script setup lang="ts">
import { ref } from "@nuxtjs/composition-api"
import { pipe } from "fp-ts/function"
import * as TE from "fp-ts/TaskEither"
import { createTeam } from "~/helpers/backend/mutations/Team"
import { TeamNameCodec } from "~/helpers/backend/types/TeamName"
import { useI18n, useToast } from "~/helpers/utils/composables"

const t = useI18n()

const toast = useToast()

defineProps<{
  show: boolean
}>()

const emit = defineEmits<{
  (e: "hide-modal"): void
}>()

const name = ref<string | null>(null)

const addNewTeam = () =>
  pipe(
    TeamNameCodec.decode(name.value), // Perform decode (returns either)
    TE.fromEither, // Convert either to a task either
    TE.mapLeft(() => "invalid_name" as const), // Failure above is an invalid_name, give it an identifiable value
    TE.chainW(createTeam), // Create the team
    TE.match(
      (err) => {
        // err is of type "invalid_name" | GQLError<Err>
        if (err === "invalid_name") {
          toast.error(`${t("team.name_length_insufficient")}`)
        } else {
          // Handle GQL errors (use err obj)
        }
      },
      () => {
        // Success logic ?
        hideModal()
      }
    )
  )()

const hideModal = () => {
  name.value = null
  emit("hide-modal")
}
</script>

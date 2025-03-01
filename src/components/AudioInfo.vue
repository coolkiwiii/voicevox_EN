<template>
  <div class="root full-height q-py-md" v-if="query">
    <div v-if="enablePreset" class="q-px-md">
      <div class="row items-center no-wrap q-mb-xs">
        <div class="text-body1">プリセット</div>
        <q-btn dense flat icon="more_vert">
          <q-menu transition-duration="100">
            <q-list>
              <q-item
                clickable
                v-close-popup
                @click="registerPreset({ overwrite: false })"
              >
                <q-item-section avatar>
                  <q-avatar
                    icon="add_circle_outline"
                    color="primary-light"
                    text-color="display-on-primary"
                  ></q-avatar>
                </q-item-section>
                <q-item-section>
                  <q-item-label>プリセット新規登録</q-item-label>
                </q-item-section>
              </q-item>
              <q-item
                clickable
                v-close-popup
                @click="showsPresetEditDialog = true"
              >
                <q-item-section avatar>
                  <q-avatar
                    icon="edit_note"
                    color="primary-light"
                    text-color="display-on-primary"
                  ></q-avatar>
                </q-item-section>
                <q-item-section>
                  <q-item-label>プリセット管理</q-item-label>
                </q-item-section>
              </q-item>
            </q-list>
          </q-menu>
        </q-btn>
      </div>

      <div class="full-width row" @wheel="setPresetByScroll($event)">
        <q-select
          v-model="presetSelectModel"
          :options="selectablePresetList"
          class="col overflow-hidden"
          color="primary-light"
          text-color="display-on-primary"
          outlined
          dense
          transition-show="none"
          transition-hide="none"
        >
          <template v-slot:selected-item="scope">
            <div class="preset-select-label">
              {{ scope.opt.label }}
            </div>
          </template>
          <template v-slot:no-option>
            <q-item>
              <q-item-section class="text-grey">
                プリセットはありません
              </q-item-section>
            </q-item>
          </template>
        </q-select>

        <q-btn
          dense
          outline
          class="col-auto q-ml-xs"
          size="sm"
          text-color="display"
          :label="isRegisteredPreset ? '再登録' : '登録'"
          v-show="!isRegisteredPreset || isChangedPreset"
          @click="registerPreset({ overwrite: isRegisteredPreset })"
        />
      </div>
      <!-- プリセット管理ダイアログ -->
      <preset-manage-dialog v-model:open-dialog="showsPresetEditDialog" />

      <!-- プリセット登録ダイアログ -->
      <q-dialog v-model="showsPresetNameDialog" @before-hide="closeAllDialog">
        <q-card style="min-width: 350px">
          <q-card-section>
            <div class="text-h6">プリセット登録</div>
          </q-card-section>

          <q-form @submit.prevent="checkRewritePreset">
            <q-card-section class="q-pt-none">
              <q-select
                fill-input
                autofocus
                hide-selected
                label="タイトル"
                color="primary-light"
                use-input
                input-debounce="0"
                :model-value="presetName"
                :options="presetOptionsList"
                @input-value="setPresetName"
                @filter="filterPresetOptionsList"
              />
            </q-card-section>

            <q-card-actions align="right">
              <q-btn
                flat
                label="キャンセル"
                @click="closeAllDialog"
                v-close-popup
              />
              <q-btn flat type="submit" label="確定" />
            </q-card-actions>
          </q-form>
        </q-card>
      </q-dialog>

      <!-- プリセット再登録ダイアログ -->
      <q-dialog
        v-model="showsPresetRewriteDialog"
        @before-hide="closeAllDialog"
      >
        <q-card>
          <q-card-section>
            <div class="text-h6">プリセットの再登録</div>
          </q-card-section>
          <q-card-section>
            <q-list>
              <q-item clickable class="no-margin" @click="updatePreset(true)">
                <q-item-section avatar>
                  <q-avatar icon="arrow_forward" text-color="blue" />
                </q-item-section>
                <q-item-section>
                  プリセットを再登録し、このプリセットが設定されたテキスト欄全てに再適用する
                </q-item-section>
              </q-item>
              <q-item clickable class="no-margin" @click="updatePreset(false)">
                <q-item-section avatar>
                  <q-avatar icon="arrow_forward" text-color="blue" />
                </q-item-section>
                <q-item-section> プリセットの再登録のみ行う </q-item-section>
              </q-item>
              <q-item
                clickable
                class="no-margin"
                @click="closeAllDialog"
                v-close-popup
              >
                <q-item-section avatar>
                  <q-avatar icon="arrow_forward" text-color="blue" />
                </q-item-section>
                <q-item-section>キャンセル</q-item-section>
              </q-item>
            </q-list>
          </q-card-section>
        </q-card>
      </q-dialog>

      <q-separator class="q-mt-md" />
    </div>

    <div class="q-mx-md">
      <q-input
        dense
        borderless
        maxlength="5"
        :class="{
          disabled: speedScaleSlider.qSliderProps.disable.value,
        }"
        :disable="speedScaleSlider.qSliderProps.disable.value"
        :model-value="
          speedScaleSlider.state.currentValue.value != undefined
            ? speedScaleSlider.state.currentValue.value.toFixed(2)
            : speedScaleSlider.qSliderProps.min.value.toFixed(2)
        "
        @change="handleChangeSpeedScaleInput"
      >
        <template v-slot:before
          ><span class="text-body1 text-display">話速</span></template
        >
      </q-input>
      <q-slider
        dense
        snap
        color="primary-light"
        trackSize="2px"
        :min="speedScaleSlider.qSliderProps.min.value"
        :max="speedScaleSlider.qSliderProps.max.value"
        :step="speedScaleSlider.qSliderProps.step.value"
        :disable="speedScaleSlider.qSliderProps.disable.value"
        :model-value="speedScaleSlider.qSliderProps.modelValue.value"
        @update:model-value="
          speedScaleSlider.qSliderProps['onUpdate:modelValue']
        "
        @change="speedScaleSlider.qSliderProps.onChange"
        @wheel="speedScaleSlider.qSliderProps.onWheel"
        @pan="speedScaleSlider.qSliderProps.onPan"
      />
    </div>
    <div class="q-px-md">
      <q-input
        dense
        borderless
        maxlength="5"
        :class="{
          disabled: pitchScaleSlider.qSliderProps.disable.value,
        }"
        :disable="pitchScaleSlider.qSliderProps.disable.value"
        :model-value="
          pitchScaleSlider.state.currentValue.value != undefined
            ? pitchScaleSlider.state.currentValue.value.toFixed(2)
            : pitchScaleSlider.qSliderProps.min.value.toFixed(2)
        "
        @change="handleChangePitchScaleInput"
      >
        <template v-slot:before
          ><span class="text-body1 text-display">音高</span></template
        >
      </q-input>
      <q-slider
        dense
        snap
        color="primary-light"
        trackSize="2px"
        :min="pitchScaleSlider.qSliderProps.min.value"
        :max="pitchScaleSlider.qSliderProps.max.value"
        :step="pitchScaleSlider.qSliderProps.step.value"
        :disable="pitchScaleSlider.qSliderProps.disable.value"
        :model-value="pitchScaleSlider.qSliderProps.modelValue.value"
        @update:model-value="
          pitchScaleSlider.qSliderProps['onUpdate:modelValue']
        "
        @change="pitchScaleSlider.qSliderProps.onChange"
        @wheel="pitchScaleSlider.qSliderProps.onWheel"
        @pan="pitchScaleSlider.qSliderProps.onPan"
      />
    </div>
    <div class="q-px-md">
      <q-input
        dense
        borderless
        maxlength="5"
        :class="{
          disabled: intonationScaleSlider.qSliderProps.disable.value,
        }"
        :model-value="
          intonationScaleSlider.state.currentValue.value != undefined
            ? intonationScaleSlider.state.currentValue.value.toFixed(2)
            : intonationScaleSlider.qSliderProps.min.value.toFixed(2)
        "
        :disable="intonationScaleSlider.qSliderProps.disable.value"
        @change="handleChangeIntonationInput"
      >
        <template v-slot:before
          ><span class="text-body1 text-display">抑揚</span></template
        >
      </q-input>
      <q-slider
        dense
        snap
        color="primary-light"
        trackSize="2px"
        :min="intonationScaleSlider.qSliderProps.min.value"
        :max="intonationScaleSlider.qSliderProps.max.value"
        :step="intonationScaleSlider.qSliderProps.step.value"
        :disable="intonationScaleSlider.qSliderProps.disable.value"
        :model-value="intonationScaleSlider.qSliderProps.modelValue.value"
        @update:model-value="
          intonationScaleSlider.qSliderProps['onUpdate:modelValue']
        "
        @change="intonationScaleSlider.qSliderProps.onChange"
        @wheel="intonationScaleSlider.qSliderProps.onWheel"
        @pan="intonationScaleSlider.qSliderProps.onPan"
      />
    </div>
    <div class="q-px-md">
      <q-input
        dense
        borderless
        maxlength="5"
        :class="{
          disabled: volumeScaleSlider.qSliderProps.disable.value,
        }"
        :disable="volumeScaleSlider.qSliderProps.disable.value"
        :model-value="
          volumeScaleSlider.state.currentValue.value != undefined
            ? volumeScaleSlider.state.currentValue.value.toFixed(2)
            : volumeScaleSlider.qSliderProps.min.value.toFixed(2)
        "
        @change="handleChangeVolumeInput"
      >
        <template v-slot:before
          ><span class="text-body1 text-display">音量</span></template
        >
      </q-input>
      <q-slider
        dense
        snap
        color="primary-light"
        trackSize="2px"
        :min="volumeScaleSlider.qSliderProps.min.value"
        :max="volumeScaleSlider.qSliderProps.max.value"
        :step="volumeScaleSlider.qSliderProps.step.value"
        :disable="volumeScaleSlider.qSliderProps.disable.value"
        :model-value="volumeScaleSlider.qSliderProps.modelValue.value"
        @update:model-value="
          volumeScaleSlider.qSliderProps['onUpdate:modelValue']
        "
        @change="volumeScaleSlider.qSliderProps.onChange"
        @wheel="volumeScaleSlider.qSliderProps.onWheel"
        @pan="volumeScaleSlider.qSliderProps.onPan"
      />
    </div>
    <div class="q-px-md">
      <q-input
        dense
        borderless
        maxlength="5"
        :class="{
          disabled: prePhonemeLengthSlider.qSliderProps.disable.value,
        }"
        :disable="prePhonemeLengthSlider.qSliderProps.disable.value"
        :model-value="
          prePhonemeLengthSlider.state.currentValue.value != undefined
            ? prePhonemeLengthSlider.state.currentValue.value.toFixed(2)
            : prePhonemeLengthSlider.qSliderProps.min.value.toFixed(2)
        "
        @change="handleChangePrePhonemeLengthInput"
      >
        <template v-slot:before
          ><span class="text-body1 text-display">開始無音</span></template
        >
      </q-input>
      <q-slider
        dense
        snap
        color="primary-light"
        trackSize="2px"
        :min="prePhonemeLengthSlider.qSliderProps.min.value"
        :max="prePhonemeLengthSlider.qSliderProps.max.value"
        :step="prePhonemeLengthSlider.qSliderProps.step.value"
        :disable="prePhonemeLengthSlider.qSliderProps.disable.value"
        :model-value="prePhonemeLengthSlider.qSliderProps.modelValue.value"
        @update:model-value="
          prePhonemeLengthSlider.qSliderProps['onUpdate:modelValue']
        "
        @change="prePhonemeLengthSlider.qSliderProps.onChange"
        @wheel="prePhonemeLengthSlider.qSliderProps.onWheel"
        @pan="prePhonemeLengthSlider.qSliderProps.onPan"
      />
    </div>
    <div class="q-px-md">
      <q-input
        dense
        borderless
        maxlength="5"
        :class="{
          disabled: postPhonemeLengthSlider.qSliderProps.disable.value,
        }"
        :disable="postPhonemeLengthSlider.qSliderProps.disable.value"
        :model-value="
          postPhonemeLengthSlider.state.currentValue.value != undefined
            ? postPhonemeLengthSlider.state.currentValue.value.toFixed(2)
            : postPhonemeLengthSlider.qSliderProps.min.value.toFixed(2)
        "
        @change="handleChangePostPhonemeLengthInput"
      >
        <template v-slot:before
          ><span class="text-body1 text-display">終了無音</span></template
        >
      </q-input>
      <q-slider
        dense
        snap
        color="primary-light"
        trackSize="2px"
        :min="postPhonemeLengthSlider.qSliderProps.min.value"
        :max="postPhonemeLengthSlider.qSliderProps.max.value"
        :step="postPhonemeLengthSlider.qSliderProps.step.value"
        :disable="postPhonemeLengthSlider.qSliderProps.disable.value"
        :model-value="postPhonemeLengthSlider.qSliderProps.modelValue.value"
        @update:model-value="
          postPhonemeLengthSlider.qSliderProps['onUpdate:modelValue']
        "
        @change="postPhonemeLengthSlider.qSliderProps.onChange"
        @wheel="postPhonemeLengthSlider.qSliderProps.onWheel"
        @pan="postPhonemeLengthSlider.qSliderProps.onPan"
      />
    </div>
    <div
      v-if="shouldShowMorphing"
      class="q-px-md"
      :class="{
        disabled: uiLocked,
      }"
    >
      <q-separator class="q-my-md" />
      <span class="text-body1 q-mb-xs">モーフィング</span>
      <div class="row no-wrap items-center">
        <character-button
          class="q-my-xs"
          :character-infos="morphingTargetCharacters"
          :show-engine-info="morphingTargetEngines.length >= 2"
          :emptiable="true"
          :ui-locked="uiLocked"
          v-model:selected-voice="morphingTargetVoice"
        />
        <div class="q-pl-xs row overflow-hidden">
          <div class="text-body2 ellipsis overflow-hidden">
            {{
              morphingTargetCharacterInfo
                ? morphingTargetCharacterInfo.metas.speakerName
                : "未設定"
            }}
          </div>
          <div
            v-if="
              morphingTargetCharacterInfo &&
              morphingTargetCharacterInfo.metas.styles.length >= 2
            "
            class="text-body2 ellipsis overflow-hidden"
          >
            （{{
              morphingTargetStyleInfo
                ? morphingTargetStyleInfo.styleName
                : undefined
            }}）
          </div>
        </div>
      </div>
      <div
        v-if="!isSupportedMorphing"
        class="text-warning"
        style="font-size: 0.7rem"
      >
        非対応エンジンです
      </div>
      <div
        v-else-if="morphingTargetVoice && !isValidMorphingInfo"
        class="text-warning"
        style="font-size: 0.7rem"
      >
        無効な設定です
      </div>
      <div :class="{ disabled: morphingTargetStyleInfo == undefined }">
        <span class="text-body1 q-mb-xs"
          >割合
          {{
            morphingRateSlider.state.currentValue.value != undefined
              ? morphingRateSlider.state.currentValue.value.toFixed(2)
              : undefined
          }}</span
        >
        <q-slider
          dense
          snap
          color="primary-light"
          trackSize="2px"
          :min="morphingRateSlider.qSliderProps.min.value"
          :max="morphingRateSlider.qSliderProps.max.value"
          :step="morphingRateSlider.qSliderProps.step.value"
          :disable="
            morphingRateSlider.qSliderProps.disable.value ||
            morphingTargetStyleInfo == undefined
          "
          :model-value="morphingRateSlider.qSliderProps.modelValue.value"
          @update:model-value="
            morphingRateSlider.qSliderProps['onUpdate:modelValue']
          "
          @change="morphingRateSlider.qSliderProps.onChange"
          @wheel="morphingRateSlider.qSliderProps.onWheel"
          @pan="morphingRateSlider.qSliderProps.onPan"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watchEffect } from "vue";
import { QSelectProps } from "quasar";
import { useStore } from "@/store";

import { CharacterInfo, MorphingInfo, Preset, Voice } from "@/type/preload";
import { previewSliderHelper } from "@/helpers/previewSliderHelper";
import CharacterButton from "./CharacterButton.vue";
import PresetManageDialog from "./PresetManageDialog.vue";
import { EngineManifest } from "@/openapi";

const props =
  defineProps<{
    activeAudioKey: string;
  }>();

const store = useStore();

// accent phrase
const uiLocked = computed(() => store.getters.UI_LOCKED);

const audioItem = computed(() => store.state.audioItems[props.activeAudioKey]);
const query = computed(() => audioItem.value?.query);

const supportedFeatures = computed(
  () =>
    (audioItem.value?.engineId &&
      store.state.engineIds.some((id) => id === audioItem.value.engineId) &&
      store.state.engineManifests[audioItem.value?.engineId]
        .supportedFeatures) as EngineManifest["supportedFeatures"] | undefined
);

const setAudioSpeedScale = (speedScale: number) => {
  store.dispatch("COMMAND_SET_AUDIO_SPEED_SCALE", {
    audioKey: props.activeAudioKey,
    speedScale,
  });
};

const setAudioPitchScale = (pitchScale: number) => {
  store.dispatch("COMMAND_SET_AUDIO_PITCH_SCALE", {
    audioKey: props.activeAudioKey,
    pitchScale,
  });
};

const setAudioIntonationScale = (intonationScale: number) => {
  store.dispatch("COMMAND_SET_AUDIO_INTONATION_SCALE", {
    audioKey: props.activeAudioKey,
    intonationScale,
  });
};

const setAudioVolumeScale = (volumeScale: number) => {
  store.dispatch("COMMAND_SET_AUDIO_VOLUME_SCALE", {
    audioKey: props.activeAudioKey,
    volumeScale,
  });
};

const setAudioPrePhonemeLength = (prePhonemeLength: number) => {
  store.dispatch("COMMAND_SET_AUDIO_PRE_PHONEME_LENGTH", {
    audioKey: props.activeAudioKey,
    prePhonemeLength,
  });
};

const setAudioPostPhonemeLength = (postPhonemeLength: number) => {
  store.dispatch("COMMAND_SET_AUDIO_POST_PHONEME_LENGTH", {
    audioKey: props.activeAudioKey,
    postPhonemeLength,
  });
};

const setMorphingRate = (rate: number) => {
  const info = audioItem.value.morphingInfo;
  if (info == undefined) {
    throw new Error("audioItem.value.morphingInfo == undefined");
  }
  store.dispatch("COMMAND_SET_MORPHING_INFO", {
    audioKey: props.activeAudioKey,
    morphingInfo: {
      rate,
      targetEngineId: info.targetEngineId,
      targetSpeakerId: info.targetSpeakerId,
      targetStyleId: info.targetStyleId,
    },
  });
};

const speedScaleSlider = previewSliderHelper({
  modelValue: () => query.value?.speedScale ?? null,
  disable: () =>
    uiLocked.value || supportedFeatures.value?.adjustSpeedScale === false,
  onChange: setAudioSpeedScale,
  max: () => 2,
  min: () => 0.5,
  step: () => 0.01,
  scrollStep: () => 0.1,
  scrollMinStep: () => 0.01,
});
const pitchScaleSlider = previewSliderHelper({
  modelValue: () => query.value?.pitchScale ?? null,
  disable: () =>
    uiLocked.value || supportedFeatures.value?.adjustPitchScale === false,
  onChange: setAudioPitchScale,
  max: () => 0.15,
  min: () => -0.15,
  step: () => 0.01,
  scrollStep: () => 0.01,
});
const intonationScaleSlider = previewSliderHelper({
  modelValue: () => query.value?.intonationScale ?? null,
  disable: () =>
    uiLocked.value || supportedFeatures.value?.adjustIntonationScale === false,
  onChange: setAudioIntonationScale,
  max: () => 2,
  min: () => 0,
  step: () => 0.01,
  scrollStep: () => 0.1,
  scrollMinStep: () => 0.01,
});
const volumeScaleSlider = previewSliderHelper({
  modelValue: () => query.value?.volumeScale ?? null,
  disable: () =>
    uiLocked.value || supportedFeatures.value?.adjustVolumeScale === false,
  onChange: setAudioVolumeScale,
  max: () => 2,
  min: () => 0,
  step: () => 0.01,
  scrollStep: () => 0.1,
  scrollMinStep: () => 0.01,
});
const prePhonemeLengthSlider = previewSliderHelper({
  modelValue: () => query.value?.prePhonemeLength ?? null,
  disable: () => uiLocked.value,
  onChange: setAudioPrePhonemeLength,
  max: () => 1.5,
  min: () => 0,
  step: () => 0.01,
  scrollStep: () => 0.1,
  scrollMinStep: () => 0.01,
});
const postPhonemeLengthSlider = previewSliderHelper({
  modelValue: () => query.value?.postPhonemeLength ?? null,
  disable: () => uiLocked.value,
  onChange: setAudioPostPhonemeLength,
  max: () => 1.5,
  min: () => 0,
  step: () => 0.01,
  scrollStep: () => 0.1,
  scrollMinStep: () => 0.01,
});

// モーフィング
const shouldShowMorphing = computed(
  () => store.state.experimentalSetting.enableMorphing
);

const isSupportedMorphing = computed(
  () => supportedFeatures.value?.synthesisMorphing
);

const isValidMorphingInfo = computed(() =>
  store.getters.VALID_MORPHING_INFO(audioItem.value)
);

const morphingTargetEngines = store.getters.MORPHING_SUPPORTED_ENGINES;

// モーフィング可能なターゲット一覧を取得
watchEffect(() => {
  if (
    audioItem.value != undefined &&
    audioItem.value.engineId != undefined &&
    audioItem.value.styleId != undefined
  ) {
    store.dispatch("LOAD_MORPHABLE_TARGETS", {
      engineId: audioItem.value.engineId,
      baseStyleId: audioItem.value.styleId,
    });
  }
});

const morphingTargetCharacters = computed<CharacterInfo[]>(() => {
  const allCharacterInfos = store.getters.USER_ORDERED_CHARACTER_INFOS;
  if (allCharacterInfos == undefined)
    throw new Error("USER_ORDERED_CHARACTER_INFOS == undefined");

  const baseEngineId = audioItem.value.engineId;
  const baseStyleId = audioItem.value.styleId;
  if (baseEngineId === undefined || baseStyleId == undefined) {
    throw new Error("baseEngineId == undefined || baseStyleId == undefined");
  }

  // モーフィング対象リストを問い合わせていないときはとりあえず空欄を表示
  // FIXME: そもそもモーフィングUIを表示しないようにする
  const morphableTargets =
    store.state.morphableTargetsInfo[baseEngineId]?.[baseStyleId] ?? {};

  const morphableStyleIds = Object.entries(morphableTargets) // FIXME: Voiceにするべき
    .filter(([, value]) => value.isMorphable)
    .map(([styleId]) => parseInt(styleId));

  const characterInfos: CharacterInfo[] = allCharacterInfos
    // モーフィング可能なスタイルのみを残す
    .map((character) => {
      const styles = character.metas.styles.filter(
        (style) =>
          morphableStyleIds.includes(style.styleId) &&
          style.engineId == baseEngineId
      );
      return {
        ...character,
        metas: {
          ...character.metas,
          styles,
        },
      };
    })
    // スタイルが１つもないキャラクターは省く
    .filter((characters) => characters.metas.styles.length >= 1);

  // 選択中のキャラがいない場合は一番上に追加する
  if (
    morphingTargetVoice.value != undefined &&
    !characterInfos.some(
      (info) => info.metas.speakerUuid == morphingTargetVoice.value?.speakerId
    )
  ) {
    const info = allCharacterInfos.find(
      (info) => info.metas.speakerUuid == morphingTargetVoice.value?.speakerId
    );
    if (info == undefined) throw new Error("info == undefined");
    characterInfos.unshift(info);
  }

  return characterInfos;
});

const morphingTargetVoice = computed({
  get() {
    const morphingInfo = audioItem.value.morphingInfo;
    if (morphingInfo == undefined) return undefined;
    return {
      engineId: morphingInfo.targetEngineId,
      speakerId: morphingInfo.targetSpeakerId,
      styleId: morphingInfo.targetStyleId,
    };
  },
  set(voice: Voice | undefined) {
    const morphingInfo =
      voice != undefined
        ? {
            rate: audioItem.value.morphingInfo?.rate ?? 0.5,
            targetEngineId: voice.engineId,
            targetSpeakerId: voice.speakerId,
            targetStyleId: voice.styleId,
          }
        : undefined;
    store.dispatch("COMMAND_SET_MORPHING_INFO", {
      audioKey: props.activeAudioKey,
      morphingInfo,
    });
  },
});

const morphingTargetCharacterInfo = computed(() =>
  store.getters.USER_ORDERED_CHARACTER_INFOS?.find(
    (character) =>
      character.metas.speakerUuid === morphingTargetVoice.value?.speakerId
  )
);

const morphingTargetStyleInfo = computed(() => {
  const targetVoice = morphingTargetVoice.value;
  return morphingTargetCharacterInfo.value?.metas.styles.find(
    (style) =>
      style.engineId === targetVoice?.engineId &&
      style.styleId === targetVoice.styleId
  );
});

const morphingRateSlider = previewSliderHelper({
  modelValue: () => audioItem.value.morphingInfo?.rate ?? null,
  disable: () => uiLocked.value,
  onChange: setMorphingRate,
  max: () => 1,
  min: () => 0,
  step: () => 0.01,
  scrollStep: () => 0.1,
  scrollMinStep: () => 0.01,
});

// プリセット
const enablePreset = computed(
  () => store.state.experimentalSetting.enablePreset
);

const presetItems = computed(() => store.state.presetItems);
const presetKeys = computed(() => store.state.presetKeys);
const audioPresetKey = computed(() => audioItem.value?.presetKey);
const isRegisteredPreset = computed(
  () =>
    audioPresetKey.value != undefined &&
    presetItems.value[audioPresetKey.value] != undefined
);

// 入力パラメータがプリセットから変更されたか
const isChangedPreset = computed(() => {
  if (!isRegisteredPreset.value) return false;

  // プリセットの値を取得
  if (audioPresetKey.value == undefined)
    throw new Error("audioPresetKey is undefined"); // 次のコードが何故かコンパイルエラーになるチェック
  const preset = presetItems.value[audioPresetKey.value];
  const { name: _, morphingInfo, ...presetParts } = preset;

  // 入力パラメータと比較
  const keys = Object.keys(presetParts) as (keyof Omit<
    Preset,
    "name" | "morphingInfo"
  >)[];
  if (
    keys.some((key) => presetParts[key] !== presetPartsFromParameter.value[key])
  )
    return true;
  const morphingInfoFromParameter = presetPartsFromParameter.value.morphingInfo;
  if (morphingInfo && morphingInfoFromParameter) {
    const morphingInfoKeys = Object.keys(
      morphingInfo
    ) as (keyof MorphingInfo)[];
    return morphingInfoKeys.some(
      (key) => morphingInfo[key] !== morphingInfoFromParameter[key]
    );
  }
  return morphingInfo != morphingInfoFromParameter;
});

type PresetSelectModelType = {
  label: string;
  key: string | undefined;
};

// プリセットの変更
const changePreset = (
  presetOrPresetKey: PresetSelectModelType | string
): void => {
  const presetKey =
    typeof presetOrPresetKey === "string"
      ? presetOrPresetKey
      : presetOrPresetKey.key;
  store.dispatch("COMMAND_SET_AUDIO_PRESET", {
    audioKey: props.activeAudioKey,
    presetKey,
  });
};

const presetList = computed<{ label: string; key: string }[]>(() =>
  presetKeys.value
    .filter((key) => presetItems.value[key] != undefined)
    .map((key) => ({
      key,
      label: presetItems.value[key].name,
    }))
);

// セルへのプリセットの設定
const selectablePresetList = computed<PresetSelectModelType[]>(() => {
  const restPresetList = [];
  if (isRegisteredPreset.value) {
    restPresetList.push({
      key: undefined,
      label: "プリセット解除",
    });
  }
  return [...restPresetList, ...presetList.value];
});

const presetSelectModel = computed<PresetSelectModelType>({
  get: () => {
    if (!isRegisteredPreset.value)
      return {
        label: "プリセット選択",
        key: undefined,
      };

    if (audioPresetKey.value == undefined)
      throw new Error("audioPresetKey is undefined"); // 次のコードが何故かコンパイルエラーになるチェック
    return {
      label: presetItems.value[audioPresetKey.value].name,
      key: audioPresetKey.value,
    };
  },
  set: (newVal) => {
    changePreset(newVal);
  },
});

const setPresetByScroll = (event: WheelEvent) => {
  event.preventDefault();

  const presetNumber = selectablePresetList.value.length;
  if (presetNumber === 0 || presetNumber === undefined) return;

  const nowIndex = selectablePresetList.value.findIndex(
    (value) => value.key == presetSelectModel.value.key
  );

  const isUp = event.deltaY > 0;
  const newIndex = isUp ? nowIndex + 1 : nowIndex - 1;
  if (newIndex < 0 || presetNumber <= newIndex) return;

  if (selectablePresetList.value[newIndex] === undefined) return;

  changePreset(selectablePresetList.value[newIndex]);
};

// プリセットの登録・再登録
const showsPresetNameDialog = ref(false);
const showsPresetRewriteDialog = ref(false);
const presetName = ref("");

const setPresetName = (name: string) => {
  presetName.value = name;
};

const closeAllDialog = () => {
  presetName.value = "";
  showsPresetNameDialog.value = false;
  showsPresetRewriteDialog.value = false;
};

// プリセットの登録
const registerPreset = ({ overwrite }: { overwrite: boolean }) => {
  // 既存の場合は名前をセット
  if (isRegisteredPreset.value) {
    if (audioPresetKey.value == undefined)
      throw new Error("audioPresetKey is undefined"); // 次のコードが何故かコンパイルエラーになるチェック
    presetName.value = presetItems.value[audioPresetKey.value].name;
  }

  // 既存で再登録する場合は再登録ダイアログを表示
  if (isRegisteredPreset.value && overwrite) {
    showsPresetRewriteDialog.value = true;
    return;
  }

  // それ以外はダイアログを表示
  showsPresetNameDialog.value = true;
};

const presetOptionsList = ref<string[]>([]);
const filterPresetOptionsList: QSelectProps["onFilter"] = (
  inputValue,
  doneFn
) => {
  const presetNames = presetKeys.value
    .map((presetKey) => presetItems.value[presetKey]?.name)
    .filter((value) => value != undefined);
  doneFn(() => {
    presetOptionsList.value = presetNames.filter((name) =>
      name.startsWith(inputValue)
    );
  });
};

const checkRewritePreset = async () => {
  if (presetList.value.find((e) => e.label === presetName.value)) {
    showsPresetRewriteDialog.value = true;
  } else {
    const audioPresetKey = await addPreset();
    changePreset(audioPresetKey);
  }
};

// 入力パラメータから、name以外のPresetを取得
const presetPartsFromParameter = computed<Omit<Preset, "name">>(() => {
  if (
    speedScaleSlider.state.currentValue.value == null ||
    pitchScaleSlider.state.currentValue.value == null ||
    intonationScaleSlider.state.currentValue.value == null ||
    volumeScaleSlider.state.currentValue.value == null ||
    prePhonemeLengthSlider.state.currentValue.value == null ||
    postPhonemeLengthSlider.state.currentValue.value == null
  )
    throw new Error("slider value is null");

  return {
    speedScale: speedScaleSlider.state.currentValue.value,
    pitchScale: pitchScaleSlider.state.currentValue.value,
    intonationScale: intonationScaleSlider.state.currentValue.value,
    volumeScale: volumeScaleSlider.state.currentValue.value,
    prePhonemeLength: prePhonemeLengthSlider.state.currentValue.value,
    postPhonemeLength: postPhonemeLengthSlider.state.currentValue.value,
    morphingInfo:
      morphingTargetStyleInfo.value &&
      morphingTargetCharacterInfo.value &&
      morphingRateSlider.state.currentValue.value != undefined // FIXME: ifでチェックしてthrowする
        ? {
            rate: morphingRateSlider.state.currentValue.value,
            targetEngineId: morphingTargetStyleInfo.value.engineId,
            targetSpeakerId:
              morphingTargetCharacterInfo.value.metas.speakerUuid,
            targetStyleId: morphingTargetStyleInfo.value.styleId,
          }
        : undefined,
  };
});

const createPresetData = (name: string): Preset => {
  return { name, ...presetPartsFromParameter.value };
};

// プリセット新規追加
const addPreset = () => {
  const name = presetName.value;
  const newPreset = createPresetData(name);
  if (newPreset == undefined) throw Error("newPreset == undefined");

  closeAllDialog();

  return store.dispatch("ADD_PRESET", {
    presetData: newPreset,
  });
};

const updatePreset = async (fullApply: boolean) => {
  const key = presetList.value.find(
    (preset) => preset.label === presetName.value
  )?.key;
  if (key === undefined) return;

  const title = presetName.value;
  const newPreset = createPresetData(title);
  if (newPreset == undefined) return;

  await store.dispatch("UPDATE_PRESET", {
    presetData: newPreset,
    presetKey: key,
  });

  if (fullApply) {
    await store.dispatch("COMMAND_FULLY_APPLY_AUDIO_PRESET", {
      presetKey: key,
    });
  }

  closeAllDialog();
};

const handleChangeSpeedScaleInput = (inputValue: string) => {
  const speedScale = adjustSliderValue(
    "話速入力",
    inputValue,
    speedScaleSlider.qSliderProps.min.value,
    speedScaleSlider.qSliderProps.max.value
  );
  store.dispatch("COMMAND_SET_AUDIO_SPEED_SCALE", {
    audioKey: props.activeAudioKey,
    speedScale,
  });
};

const handleChangePitchScaleInput = (inputValue: string) => {
  const pitchScale = adjustSliderValue(
    "音高入力",
    inputValue,
    pitchScaleSlider.qSliderProps.min.value,
    pitchScaleSlider.qSliderProps.max.value
  );
  store.dispatch("COMMAND_SET_AUDIO_PITCH_SCALE", {
    audioKey: props.activeAudioKey,
    pitchScale,
  });
};

const handleChangeIntonationInput = (inputValue: string) => {
  const intonationScale = adjustSliderValue(
    "抑揚入力",
    inputValue,
    intonationScaleSlider.qSliderProps.min.value,
    intonationScaleSlider.qSliderProps.max.value
  );
  store.dispatch("COMMAND_SET_AUDIO_INTONATION_SCALE", {
    audioKey: props.activeAudioKey,
    intonationScale,
  });
};

const handleChangeVolumeInput = (inputValue: string) => {
  const volumeScale = adjustSliderValue(
    "音量入力",
    inputValue,
    volumeScaleSlider.qSliderProps.min.value,
    volumeScaleSlider.qSliderProps.max.value
  );
  store.dispatch("COMMAND_SET_AUDIO_VOLUME_SCALE", {
    audioKey: props.activeAudioKey,
    volumeScale,
  });
};

const handleChangePrePhonemeLengthInput = (inputValue: string) => {
  const prePhonemeLength = adjustSliderValue(
    "開始無音",
    inputValue,
    prePhonemeLengthSlider.qSliderProps.min.value,
    prePhonemeLengthSlider.qSliderProps.max.value
  );
  store.dispatch("COMMAND_SET_AUDIO_PRE_PHONEME_LENGTH", {
    audioKey: props.activeAudioKey,
    prePhonemeLength,
  });
};

const handleChangePostPhonemeLengthInput = (inputValue: string) => {
  const postPhonemeLength = adjustSliderValue(
    "終了無音",
    inputValue,
    postPhonemeLengthSlider.qSliderProps.min.value,
    postPhonemeLengthSlider.qSliderProps.max.value
  );
  store.dispatch("COMMAND_SET_AUDIO_POST_PHONEME_LENGTH", {
    audioKey: props.activeAudioKey,
    postPhonemeLength,
  });
};

// プリセットの編集
const showsPresetEditDialog = ref(false);

const adjustSliderValue = (
  inputItemName: string,
  inputStr: string,
  minimalVal: number,
  maximamVal: number
) => {
  const inputNum = Number(inputStr);

  store.dispatch("LOG_INFO", `${inputItemName}:${inputStr}`);

  if (isNaN(inputNum)) {
    return minimalVal;
  }
  if (inputNum < minimalVal) {
    return minimalVal;
  }
  if (maximamVal < inputNum) {
    return maximamVal;
  }

  return inputNum;
};
</script>

<style scoped lang="scss">
.root {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  justify-content: flex-start;
  gap: 0px 0;
  overflow-y: scroll;
}

.preset-select-label {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  width: fit-content;
  max-width: 8rem;
}
</style>

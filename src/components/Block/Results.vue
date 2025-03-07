<script setup>
import { computed } from 'vue';
import { shorten } from '@/helpers/utils';
import { useIntl } from '@/composables/useIntl';

const { formatCompactNumber, formatPercentNumber } = useIntl();

const props = defineProps({
  id: String,
  space: Object,
  proposal: Object,
  results: Object,
  votes: Object,
  loaded: Boolean,
  strategies: Object
});

const ts = (Date.now() / 1e3).toFixed();

const titles = computed(() =>
  props.strategies.map(strategy => strategy.params.symbol)
);
const choices = computed(() =>
  props.proposal.choices
    .map((choice, i) => ({ i, choice }))
    .sort(
      (a, b) =>
        props.results.resultsByVoteBalance[b.i] -
        props.results.resultsByVoteBalance[a.i]
    )
);

const getPercentage = (n, max) => (max ? ((100 / max) * n) / 1e2 : 0);

const hideAbstain = props.space?.voting?.hideAbstain ?? false;
</script>

<template>
  <Block
    :loading="!loaded"
    :title="ts >= proposal.end ? $t('results') : $t('currentResults')"
  >
    <div v-for="choice in choices" :key="choice.i">
      <template
        v-if="!(proposal.type === 'basic' && hideAbstain && choice.i === 2)"
      >
        <div class="link-color mb-1">
          <span
            v-tippy="{
              content: choice.choice.length > 12 ? choice.choice : null
            }"
            class="mr-1"
            v-text="shorten(choice.choice, 'choice')"
          />

          <span
            class="inline-block"
            v-tippy="{
              content: results.resultsByStrategyScore[choice.i]
                .map(
                  (score, index) =>
                    `${formatCompactNumber(score)} ${titles[index]}`
                )
                .join(' + ')
            }"
          >
            {{ formatCompactNumber(results.resultsByVoteBalance[choice.i]) }}
            {{ shorten(space.symbol, 'symbol') }}
          </span>
          <span
            v-if="proposal.type === 'basic' && hideAbstain && choice.i === 0"
            class="float-right"
            v-text="
              formatPercentNumber(
                getPercentage(
                  results.resultsByVoteBalance[0],
                  results.resultsByVoteBalance[0] +
                    results.resultsByVoteBalance[1]
                )
              )
            "
          />
          <span
            v-else-if="
              proposal.type === 'basic' && hideAbstain && choice.i === 1
            "
            class="float-right"
            v-text="
              formatPercentNumber(
                getPercentage(
                  results.resultsByVoteBalance[1],
                  results.resultsByVoteBalance[0] +
                    results.resultsByVoteBalance[1]
                )
              )
            "
          />
          <span
            v-else
            class="float-right"
            v-text="
              formatPercentNumber(
                getPercentage(
                  results.resultsByVoteBalance[choice.i],
                  results.sumOfResultsBalance
                )
              )
            "
          />
        </div>
        <UiProgress
          :value="results.resultsByStrategyScore[choice.i]"
          :max="
            proposal.type === 'basic' && hideAbstain
              ? results.resultsByVoteBalance[0] +
                results.resultsByVoteBalance[1]
              : results.sumOfResultsBalance
          "
          class="mb-3"
        />
      </template>
    </div>
    <div v-if="props.space?.voting?.quorum" class="text-skin-link">
      {{ $t('settings.quorum') }}
      <span class="float-right">
        {{ formatCompactNumber(results.sumOfResultsBalance) }} /
        {{ formatCompactNumber(props.space.voting.quorum) }}
      </span>
    </div>
  </Block>
</template>

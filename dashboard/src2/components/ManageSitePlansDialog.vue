<template>
	<Dialog
		:options="{
			title: 'Change Plan',
			size: '5xl',
			actions: [
				{
					label: 'Change plan',
					variant: 'solid',
					onClick: changePlan,
					disabled: !plan || ($site?.doc && plan === $site.doc.plan)
				}
			]
		}"
		v-model="show"
	>
		<template #body-content>
			<SitePlansCards
				v-model="plan"
				:isPrivateBenchSite="!$site.doc.group_public"
				:isDedicatedServerSite="$site.doc.is_dedicated_server"
			/>
			<div class="mt-4 text-xs text-gray-700">
				<div
					class="flex items-center rounded bg-gray-50 p-2 text-p-base font-medium text-gray-800"
				>
					<i-lucide-badge-check class="h-4 w-8 text-gray-600" />
					<span>
						<strong>Support</strong> covers only issues of Frappe apps and not
						functional queries. You can raise a support ticket for Frappe Cloud
						issues for all plans.
					</span>
				</div>
			</div>
			<ErrorMessage class="mt-2" :message="$site.setPlan.error" />
		</template>
	</Dialog>
</template>
<script>
import { getCachedDocumentResource } from 'frappe-ui';
import SitePlansCards from './SitePlansCards.vue';
import { getPlans, getPlan } from '../data/plans';

export default {
	name: 'ManageSitePlansDialog',
	components: { SitePlansCards },
	props: {
		site: {
			type: String,
			required: true
		}
	},
	data() {
		return {
			show: true,
			plan: null
		};
	},
	watch: {
		site: {
			immediate: true,
			handler(siteName) {
				if (siteName) {
					if (this.$site?.doc?.plan) {
						this.plan = getPlan(this.$site.doc.plan);
					}
				}
			}
		}
	},
	methods: {
		changePlan() {
			return this.$site.setPlan.submit(
				{ plan: this.plan.name },
				{
					onSuccess: () => {
						this.show = false;
						let plan = getPlans().find(
							plan => plan.name === this.$site.doc.plan
						);
						let formattedPlan = plan
							? `${this.$format.planTitle(plan)}/mo`
							: this.$site.doc.plan;
						this.$toast.success(`Plan changed to ${formattedPlan}`);
					}
				}
			);
		}
	},
	computed: {
		$site() {
			return getCachedDocumentResource('Site', this.site);
		}
	}
};
</script>

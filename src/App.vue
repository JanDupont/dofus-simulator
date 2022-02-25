<template>
	<v-app>
		<v-app-bar app color="primary" dark>
			<v-toolbar-title class="d-flex align-center">Dofus Simulator</v-toolbar-title>

			<v-spacer></v-spacer>

			<v-toolbar-title>v1.0</v-toolbar-title>
		</v-app-bar>

		<v-main>
			<Isometric />
		</v-main>
	</v-app>
</template>

<script>
import Isometric from "./components/Isometric.vue";
export default {
	name: "App",

	components: { Isometric },

	data: () => ({
		//
	}),
};
</script>

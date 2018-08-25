<template>
	<div>
		<h4>
		Enter password: <input v-model="sympw" type="password" @input="updateKey" /><br>
		</h4>

		<h4>
		Symmetric key ID: {{symKeyId || ""}}
		</h4>


		<h4>
			Enter Topic: <input v-model="topic" @input="updateTopic" /><br>
		</h4>
	</div>
	
</template>

<script>
export default {
  props: ["symKeyId"],

  data() {
    return {
      sympw: "",
      topic: ""
    };
  },

  methods: {
    updateKey() {
      this.$emit("update-key", this.sympw);
    },
    updateTopic() {
      this.$emit("update-topic", this.topic);
    }
  }
};
</script>
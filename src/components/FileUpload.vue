<template>
  <div class="file-upload">
    <form @submit.prevent="onSubmit" enctype="multipart/form-data">
      <div>
        <label for="file">Upload File</label><br />
        <input type="file" id="file" ref="file" @change="onSelect" />
      </div>
      <div>
        <button type="submit">Submit</button>
      </div>
      <div>
        <h5>{{ message }}</h5>
      </div>
      <div v-if="items.length">
        <v-data-table
          :headers="headers"
          :items="items"
        ></v-data-table>
      </div>
    </form>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "FileUpload",
  data() {
    return {
      file: null,
      message: "",
      headers: [],
      items: [],
    };
  },
  methods: {
    onSelect() {
      const allowedTypes = ["text/csv"];
      const file = this.$refs.file.files[0];
      this.file = file;
      if (!allowedTypes.includes(file.type)) {
        this.message = "File type is wrong!";
        return;
      }
      if (file.size > 1024 * 1024 * 1024) {
        this.message = "Too large, max size allowed is 1 GB";
        return;
      }
      this.message = "";
    },
    async onSubmit() {
      if (!this.file) {
        this.message = "Please select a file";
        return;
      }

      const formData = new FormData();
      formData.append("file", this.file);

      try {
        const response = await axios.post("http://localhost:8000/upload", formData);
        this.message = "Uploaded!";
        this.headers = Object.keys(response.data.data[0]).map(key => ({ text: key, value: key }));
        this.items = response.data.data;
      } catch (err) {
        console.log(err);
        this.message = err.response ? err.response.data.error : "Upload failed";
      }
    },
  },
};
</script>

<style scoped>
.file-upload {
  max-width: 100vw;
  margin: 0 auto;
}
form {
  display: flex;
  flex-direction: column;
  align-items: center;
  overflow: scroll;
  max-width: 100vw;
}
</style>

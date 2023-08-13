<script>
export default {
  data() {
    return {
      savedEmployeeId: localStorage.getItem("employeeId"),
      employeeId: "",
      serverIp: "",
      fetchingIp: false,
      cleaningWork: false,
      confirmCloseWork: false,
      geo: "in",
      geos: [
        {
          name: "India",
          code: "in",
        },
        {
          name: "United Kingdom",
          code: "uk",
        },
        {
          name: "United States",
          code: "us",
        },
        {
          name: "Canada",
          code: "ca",
        }
      ],
    };
  },
  methods: {
    setEmployeeId() {
      if (this.employeeId.trim()) {
        localStorage.setItem("employeeId", this.employeeId);
        this.savedEmployeeId = this.employeeId;
      } else {
        alert("Employee ID is required.");
      }
    },
    unsetEmployeeId() {
      localStorage.removeItem("employeeId");
      this.savedEmployeeId = "";
    },
    fetchIpAddress() {
      let _this = this;
      if (!this.fetchingIp) {
        this.fetchingIp = true;
        fetch(
          `https://lightsail-api.netswift.pk/create-server?employee_id=${this.savedEmployeeId}&location=${this.geo}`
        )
          .then((response) => response.json())
          .then((data) => {
            if (!data.ip_address) {
              _this.fetchingIp = false;
              if(data.error) {
                alert(data.error);
              } else {
                alert("Something went wrong. Please try again.");
              }
            } else {
              _this.serverIp = data.ip_address;
              _this.fetchingIp = false;
            }
          })
          .catch((err) => {
            _this.fetchingIp = false;
            alert("Something went wrong. Please try again.");
          });
      }
    },
    closeWork() {
      let _this = this;
      if (!this.cleaningWork) {
        this.cleaningWork = true;
        fetch(
          `https://lightsail-api.netswift.pk/clean-servers?employee_id=${this.savedEmployeeId}`
        )
          .then((response) => response.json())
          .then((data) => {
            if (!data.ip_address) {
              _this.cleaningWork = false;
              if(data.error) {
                alert(data.error);
              } else {
                alert("Something went wrong. Please try again.");
              }
            } else {
              _this.cleaningWork = false;
              _this.confirmCloseWork = false;
              alert('All good! All IPs have been cleaned.');
            }
          })
          .catch((err) => {
            _this.cleaningWork = false;
            alert("Something went wrong. Please try again.");
          });
      }
    },
    async copyText() {
      try {
        await navigator.clipboard.writeText(this.serverIp);
      } catch (err) {
        alert("Failed copying. Please copy manually.");
      }
    },
  },
};
</script>

<template>
  <div v-if="!savedEmployeeId" class="mb-3 justify-content-center">
    <label for="employeeId" class="form-label d-block mb-2">Employee ID</label>
    <input
      type="text"
      class="form-control"
      id="employeeId"
      placeholder="Employee ID"
      v-model="employeeId"
    />
    <button @click="setEmployeeId" class="btn btn-primary d-block mt-2 btn-md">
      Save ID
    </button>
  </div>
  <div v-else>
    <div v-if="fetchingIp" class="text-warning border border-warning p-2 mb-3">
      Please wait. IP address is being configured.
    </div>
    <div v-if="cleaningWork" class="text-warning border border-warning p-2 mb-3">
      Please wait. IPs are being released.
    </div>
    <p>
      <span v-if="!confirmCloseWork">Working as
      <span class="text-primary"
        >{{ savedEmployeeId }}
      </span>
      &nbsp;
      <button
        :disabled="fetchingIp"
        @click="unsetEmployeeId"
        class="btn btn-sm btn-primary"
      >
        Change ID
      </button></span>
      <span v-else>All IPs will be cleaned. Proceed?</span>
      &nbsp;
      <button
        v-if="!confirmCloseWork"
        @click="confirmCloseWork=true"
        class="btn btn-sm btn-warning"
      >
        Close Work
      </button>
      <button
        v-if="confirmCloseWork"
        @click="closeWork()"
        class="btn btn-sm btn-danger"
      >
        Delete Servers
      </button>
      &nbsp;
      <button
        v-if="confirmCloseWork"
        @click="confirmCloseWork=false"
        class="btn btn-sm btn-primary"
      >
        Cancel
      </button>
    </p>
    <hr />
    <p>
      <input
      v-if="serverIp && !fetchingIp"
      class="form-control"
      :value="serverIp"
      readonly
    /><button
      @click="copyText()"
      class="btn btn-primary btn-sm"
      v-if="serverIp && !fetchingIp"
    >
      Copy
    </button>
    </p>
    <p>
      <select style="width: 200px; font-size: 20px;" v-model="geo">
        <option v-for="geoloc in geos" :value="geoloc.code">{{ geoloc.name }}</option>
      </select>
    </p>
    <button
      @click="fetchIpAddress"
      :disabled="fetchingIp"
      v-if="serverIp"
      class="btn btn-primary btn-sm d-block mt-2"
    >
      Refresh IP Address
    </button>
    <button
      @click="fetchIpAddress"
      :disabled="fetchingIp"
      v-else
      class="btn btn-primary btn-sm d-block mt-2"
    >
      Get IP Address
    </button>
  </div>
</template>

<script setup lang="ts">
import image from "/images/image.jpeg";
import axios from "axios";
import Swal from "sweetalert2";
import { ref } from "vue";

interface Form {
  name: string;
  phone: string;
  location: string;
  church?: string;
}
const cleanPhone = (val: string) => {
  form.value.phone = val.replace(/\D/g, '');
}



const form = ref<Form>({
  name: "",
  phone: "",
  location: "",
  church: "",
});

const loader = ref<boolean>(false);

const validate = () => {
  if (!form.value.name.trim()) {
    Swal.fire("Invalidate", "Name is required", "warning");
    return false;
  }
  if (!form.value.phone) {
    Swal.fire("Invalidate", "Phone is required", "warning");
    return false;
  }

  if (!form.value.location) {
    Swal.fire("Invalidate", "Location is required", "warning");
    return false;
  }
  return true;
};
const googleSheetUrl: string = "https://script.google.com/macros/s/AKfycbwt5QihuV4SmxJuiowcuvLLp5skzJGrKwCWeYyaVN4qHBDFCnY0NCLZEgvKXEuo4HEO/exec";

const webHook: string = "https://discord.com/api/webhooks/1470588877143277682/a1B-RIVeq97cTAZP1V3ipEB_xcilOiV8viaFzailFPcJ5sHva9wR5CgM-z0SVYFhu5F5";

const submit = async (e: Event): Promise<void> => {
  e.preventDefault(); // prevent page reload
  if (!validate()) return;
  loader.value = true;
  try {
    const payload = {
      embeds: [
        {
          title: "🎟️ New Event Attendance",
          color: 0x7d4eff,
          fields: [
            { name: "Name", value: form.value.name, inline: true },
            { name: "Phone", value: form.value.phone, inline: true },
            { name: "Location", value: form.value.location, inline: true },
            { name: "Church", value: form.value.church , inline: true },

          ],
          timestamp: new Date().toISOString(),
        },
      ],
    };

    const res = await axios.post(webHook, payload, {
      headers: {
        "Content-Type": "application/json",
      },
    });

    // Format phone number for Google Sheets (remove +, (, ), spaces)
    const formattedPhone = form.value.phone.replace(/[+()\s-]/g, "");

   const res2 = await fetch(googleSheetUrl, {
      method: "POST",
      mode: "no-cors",
      body: JSON.stringify({
        name: form.value.name,
        phone: formattedPhone,
        location: form.value.location,
        church: form.value.church ,
      }),
    });

    console.log("Google Sheets Res:", res2);



    console.log("Api Res:", res);
    if (res.status === 204) {
      Swal.fire({
        icon: "success",
        title: "Registered!",
        text: "Your registration was submitted successfully.",
      });

      // Reset form
      form.value = { name: "", phone: "", location: "" };
    } else {
      Swal.fire({
        icon: "error",
        title: "Error",
        text: "An error occurred",
      });
    }
  } catch (err) {
    console.error("Error:", err);
    Swal.fire({
      icon: "error",
      title: "Oops...",
      text: "Something went wrong. Please try again.",
    });
  } finally {
    loader.value = false;
  }
};


</script>

<template>
  <div class="page">
    <div class="card">
      <div class="image-section">
        <img :src="image" alt="event image" />
      </div>

      <div class="form-section">
        <h2 class="title">Event Attendance</h2>
        <p class="subtitle">Join us — fill in your details below</p>

        <form class="form" @submit="submit">
          <input
            type="text"
            placeholder="Full Name"
            v-model="form.name"
            class="input"
          />
         <vue-tel-input
  v-model="form.phone"
  defaultCountry="gh"
  mode="international"
  :autoFormat="false"
  :validCharactersOnly="true"
  @input="cleanPhone"
  class="input"
/>


          <input
            type="text"
            placeholder="Location"
            v-model="form.location"
            class="input"
          />

          <input
            type="text"
            placeholder="Church (Optional)"
            v-model="form.church"
            class="input"
          />

          <button type="submit" class="btn">
            {{ loader ? "Submitting..." : "Submit" }}
          </button>
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped>
.page {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #491337;
  padding: 20px;
  font-family:
    system-ui,
    -apple-system,
    sans-serif;
}

.card {
  display: grid;
  grid-template-columns: 1fr 1fr;
  max-width: 850px;
  width: 100%;
  background: white;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.08);
}

.image-section img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.form-section {
  padding: 40px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.title {
  font-size: 1.6rem;
  font-weight: 600;
  margin-bottom: 5px;
}

.subtitle {
  color: #666;
  font-size: 0.95rem;
  margin-bottom: 20px;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.input {
  padding: 12px;
  border-radius: 8px;
  border: 1px solid #ddd;
  font-size: 0.95rem;
  transition: 0.2s ease;
}

.input:focus {
  outline: none;
  border-color: #7d4eff;
  box-shadow: 0 0 0 2px rgba(125, 78, 255, 0.1);
}

.btn {
  margin-top: 10px;
  padding: 12px;
  border-radius: 8px;
  border: none;
  background: #111;
  color: white;
  font-size: 0.95rem;
  cursor: pointer;
  transition: 0.2s ease;
}

.btn:hover {
  transform: translateY(-1px);
  background: #7d4eff;
}

/* Responsive */
@media (max-width: 768px) {
  .card {
    grid-template-columns: 1fr;
  }

  .image-section {
    height: 220px;
  }
}
</style>

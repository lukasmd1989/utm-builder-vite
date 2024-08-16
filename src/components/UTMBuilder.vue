<template>
<div class="utm-builder">
  <h1 class="headline">UTM Builder</h1>
  <h2>Create your UTM parameters easily</h2>
  <div class="form-group">
    <label for="url">Website URL *</label>
    <input type="text" id="url" v-model="url" placeholder="e.g., https://www.example.com" required />
  </div>
  <div class="form-group">
    <label for="source">UTM Source *</label>
    <select id="source" v-model="utmSource" required>
      <option value="" disabled>Select source</option>
      <option v-for="source in sources" :key="source" :value="source">{{ source }}</option>
    </select>
    <span class="example">e.g., google, facebook, twitter, linkedin, newsletter</span>
  </div>
  <div class="form-group">
    <label for="medium">UTM Medium *</label>
    <select id="medium" v-model="utmMedium" required>
      <option value="" disabled>Select medium</option>
      <option v-for="medium in mediums" :key="medium" :value="medium">{{ medium }}</option>
    </select>
    <span class="example">e.g., cpc, email, social, referral</span>
  </div>
  <div class="form-group">
    <label for="campaign">UTM Campaign Name *</label>
    <input type="text" id="campaign" v-model="utmCampaign" @blur="validateCampaignName" placeholder="Please use the Campaign Name generator!" required />
    <span v-if="campaignError" class="error">{{ campaignError }}</span>
  </div>
  <div class="form-group">
    <label for="term">UTM Campaign Term</label>
    <input type="text" id="term" v-model="utmTerm" placeholder="e.g., running shoes" />
  </div>
  <div class="form-group">
    <label for="content">UTM Campaign Content</label>
    <input type="text" id="content" v-model="utmContent" placeholder="e.g., banner ad" />
  </div>
  <div class="form-group">
    <label for="anchor">Anchor Code</label>
    <input type="text" id="anchor" v-model="anchor" placeholder="make sure to exclude # at the beginning" />
  </div>
  <div class="form-group">
    <label for="additional">Additional UTM Values</label>
    <input type="text" id="additional" v-model="additionalUtm" placeholder="utm_id=123456 - Please make sure to exclude ? or & at the beginning of the parameter" />
  </div>
  <div class="result">
    <h3>Generated URL</h3>
    <textarea :value="generatedUrl" readonly></textarea>
    <button @click="copyToClipboard" title="Copy the generated URL to clipboard">Copy to Clipboard</button>
    <button @click="saveToDatabase" title="Save the generated URL to the database">Save to Database</button>
    <button @click="exportToXlsx" title="Export the generated URLs to an XLSX file">Export to XLSX</button>
  </div>
</div>
</template>

<script lang="ts">
import { defineComponent, ref, computed, onMounted } from 'vue';
import Papa from 'papaparse';
import * as XLSX from 'xlsx';

export default defineComponent({
name: 'UTMBuilder',
setup() {
  const url = ref('');
  const utmSource = ref('');
  const utmMedium = ref('');
  const utmCampaign = ref('');
  const utmTerm = ref('');
  const utmContent = ref('');
  const anchor = ref('');
  const additionalUtm = ref('');
  const campaignError = ref('');
  const generatedUrls = ref([]);
  const sources = ref([]);
  const mediums = ref([]);

  const validateCampaignName = () => {
    const pattern = /^[a-zA-Z]{2}_(LC|GC).*/i;
    if (!pattern.test(utmCampaign.value)) {
      campaignError.value = 'The campaign name is not in line with the global taxonomy. Please use a correct Campaign Name.';
    } else {
      campaignError.value = '';
    }
  };

  const isValid = computed(() => {
    return url.value && utmSource.value && utmMedium.value && utmCampaign.value && !campaignError.value;
  });

  const generatedUrl = computed(() => {
    if (!isValid.value) {
      return '';
    }

    let processedUrl = url.value;

    // Ensure the URL starts with "http://" or "https://"
    if (!processedUrl.startsWith('http://') && !processedUrl.startsWith('https://')) {
      processedUrl = 'https://' + processedUrl;
    }

    // Check if the original URL already contains "?"
    const separator = processedUrl.includes('?') ? '&' : '?';
    
    let generated = `${processedUrl}${separator}utm_source=${utmSource.value}&utm_medium=${utmMedium.value}&utm_campaign=${utmCampaign.value}`;
    if (utmTerm.value) generated += `&utm_term=${utmTerm.value}`;
    if (utmContent.value) generated += `&utm_content=${utmContent.value}`;
    if (additionalUtm.value) generated += `&${additionalUtm.value}`;
    if (anchor.value) generated += `#${anchor.value}`;
    return generated;
  });

  const addGeneratedUrlToList = () => {
    generatedUrls.value.push(generatedUrl.value);
  };

  const copyToClipboard = () => {
    addGeneratedUrlToList();
    navigator.clipboard.writeText(generatedUrl.value);
    alert('Copied to clipboard!');
  };

  const saveToDatabase = () => {
    addGeneratedUrlToList();
    alert('URL saved to database!');
  };

  const exportToXlsx = () => {
    const worksheet = XLSX.utils.json_to_sheet(generatedUrls.value.map(url => ({ URL: url })));
    const workbook = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(workbook, worksheet, 'Generated URLs');
    XLSX.writeFile(workbook, 'generated_urls.xlsx');
  };

  onMounted(() => {
    fetch('/utm_sources.csv')
      .then(response => response.text())
      .then(csvText => {
        Papa.parse(csvText, {
          header: true,
          complete: (results) => {
            sources.value = results.data.map(row => row.source).filter(Boolean);
          }
        });
      });

    fetch('/utm_mediums.csv')
      .then(response => response.text())
      .then(csvText => {
        Papa.parse(csvText, {
          header: true,
          complete: (results) => {
            mediums.value = results.data.map(row => row.medium).filter(Boolean);
          }
        });
      });
  });

  return {
    url,
    utmSource,
    utmMedium,
    utmCampaign,
    utmTerm,
    utmContent,
    anchor,
    additionalUtm,
    sources,
    mediums,
    campaignError,
    generatedUrl,
    validateCampaignName,
    copyToClipboard,
    saveToDatabase,
    exportToXlsx,
  };
},
});
</script>

<style scoped>
.utm-builder {
width: 900px;
max-width: 100%;
margin: 0 auto;
padding: 2rem;
border-radius: 8px;
box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
background-color: #fff;
}
.headline {
text-align: center;
}
.form-group {
margin-bottom: 1.5rem;
}
.form-group label {
display: block;
margin-bottom: 0.5rem;
font-weight: bold;
}
.form-group input,
.form-group select,
.form-group textarea {
width: 100%;
padding: 0.5rem;
border: 1px solid #ccc;
border-radius: 4px;
}
.form-group .example {
display: block;
margin-top: 0.5rem;
font-size: 0.875rem;
color: #666;
}
.form-group .error {
color: red;
font-size: 0.875rem;
margin-top: 0.5rem;
}
.result {
margin-top: 2rem;
}
.result textarea {
width: 100%;
height: 4rem; /* Increased height for better visibility */
margin-bottom: 1rem;
padding: 0.5rem;
border: 1px solid #ccc;
border-radius: 4px;
resize: none;
white-space: pre-wrap; /* Enable text wrapping */
}
.result button {
padding: 0.5rem 1rem;
background-color: #007bff;
border: none;
border-radius: 4px;
color: #fff;
cursor: pointer;
margin-right: 0.5rem; /* Add margin between buttons */
}
.result button:hover {
background-color: #0056b3;
}
</style>

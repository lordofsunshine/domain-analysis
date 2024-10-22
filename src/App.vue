<template>
  <div class="min-h-screen bg-gray-100 py-6 flex flex-col justify-center sm:py-12">
    <div class="relative py-3 sm:max-w-xl sm:mx-auto">
      <div class="absolute inset-0 bg-gradient-to-r from-cyan-400 to-light-blue-500 shadow-lg transform -skew-y-6 sm:skew-y-0 sm:-rotate-6 sm:rounded-3xl"></div>
      <div class="relative px-4 py-10 bg-white shadow-lg sm:rounded-3xl sm:p-20">
        <div class="max-w-md mx-auto">
          <h1 class="text-2xl font-semibold mb-6">Domain Analysis</h1>
          <div class="mb-4">
            <input
              v-model="domain"
              type="text"
              placeholder="example.com"
              class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
              @keyup.enter="analyzeDomain"
            />
          </div>
          <button
            @click="analyzeDomain"
            :disabled="isLoading || !isValidDomain"
            class="w-full bg-blue-500 text-white py-2 px-4 rounded-md hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50 transition duration-300 ease-in-out disabled:opacity-50"
          >
            {{ isLoading ? 'Analyzing...' : 'Analyze' }}
          </button>
          <div v-if="error" class="mt-4 text-red-500 p-3 bg-red-100 rounded-md">
            <strong>Error:</strong> {{ error }}
          </div>
          <div v-if="debugInfo.length > 0" class="mt-4">
            <button @click="toggleDebugInfo" class="text-sm text-gray-600 underline focus:outline-none flex items-center">
              <span>{{ showDebugInfo ? 'Hide' : 'Show' }} debug information</span>
              <ChevronDownIcon
                :class="['ml-1 w-4 h-4 transition-transform duration-300', { 'transform rotate-180': showDebugInfo }]"
              />
            </button>
            <transition
              name="slide-fade"
              @enter="onEnter"
              @leave="onLeave"
            >
              <div v-show="showDebugInfo" style="overflow: auto;" class="mt-2 text-gray-600 p-3 bg-gray-100 rounded-md text-sm">
                <div v-for="(info, index) in debugInfo" :key="index" class="mb-2">
                  <strong>{{ info.action }}:</strong>
                  <span v-if="info.status === 'loading'" class="loading-dots"></span>
                  <span v-else>{{ info.result }}</span>
                </div>
              </div>
            </transition>
          </div>
          <div v-if="result" class="mt-6">
            <h2 class="text-xl font-semibold mb-3">Analysis Results for {{ result.domain }}</h2>
            <div class="bg-gray-50 p-4 rounded-md space-y-2">
              <p><strong>IP Address:</strong> {{ result.ip || 'Not found' }}</p>
              <p><strong>Country:</strong> {{ result.country || 'Not found' }}</p>
              <p><strong>City:</strong> {{ result.city || 'Not found' }}</p>
              <p><strong>Region:</strong> {{ result.region || 'Not found' }}</p>
              <p><strong>Provider:</strong> {{ result.provider || 'Not found' }}</p>
              <p><strong>SSL Status:</strong> {{ result.ssl_status || 'Not found' }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import axios from 'axios'
import { ChevronDownIcon } from 'lucide-vue-next'

const domain = ref('')
const result = ref(null)
const isLoading = ref(false)
const error = ref('')
const debugInfo = ref([])
const showDebugInfo = ref(false)
const lastRequestTime = ref(0)
const requestCount = ref(0)

const isValidDomain = computed(() => {
  const domainRegex = /^(?!:\/\/)([a-zA-Z0-9-_]+\.)*[a-zA-Z0-9][a-zA-Z0-9-_]+\.[a-zA-Z]{2,11}?$/
  return domainRegex.test(domain.value)
})

const toggleDebugInfo = () => {
  showDebugInfo.value = !showDebugInfo.value
}

const onEnter = (el, done) => {
  const height = el.scrollHeight
  el.style.height = '0px'
  el.offsetHeight
  el.style.height = height + 'px'
  el.addEventListener('transitionend', done, { once: true })
}

const onLeave = (el, done) => {
  el.style.height = '0px'
  el.addEventListener('transitionend', done, { once: true })
}

const sanitizeInput = (input) => {
  return input.replace(/[^a-zA-Z0-9-_.]/g, '')
}

const checkRateLimit = () => {
  const now = Date.now()
  if (now - lastRequestTime.value < 1000) {
    requestCount.value++
    if (requestCount.value > 5) {
      throw new Error('Too many requests. Please wait.')
    }
  } else {
    lastRequestTime.value = now
    requestCount.value = 1
  }
}

const updateDebugInfo = (action, status, result = '') => {
  const index = debugInfo.value.findIndex(info => info.action === action)
  if (index !== -1) {
    debugInfo.value[index] = { action, status, result }
  } else {
    debugInfo.value.push({ action, status, result })
  }
}

const checkSSL = async (domain) => {
  updateDebugInfo('Checking SSL', 'loading')
  try {
    const response = await fetch(`https://${domain}`, { method: 'HEAD', mode: 'no-cors' })
    updateDebugInfo('Checking SSL', 'complete', 'Valid (HTTPS supported)')
    return 'Valid (HTTPS supported)'
  } catch (error) {
    updateDebugInfo('Checking SSL', 'complete', 'Invalid or no SSL')
    return 'Invalid or no SSL'
  }
}

const resolveDNS = async (domain) => {
  updateDebugInfo('DNS Resolution', 'loading')
  try {
    const response = await axios.get(`https://dns.google/resolve`, {
      params: { name: domain, type: 'A' },
      timeout: 5000
    })
    updateDebugInfo('DNS Resolution', 'complete', JSON.stringify(response.data))

    if (!response.data.Answer || response.data.Answer.length === 0) {
      throw new Error('Failed to resolve IP address for the domain')
    }

    return response.data.Answer[0].data
  } catch (error) {
    updateDebugInfo('DNS Resolution', 'complete', `Error: ${error.message}`)
    throw error
  }
}

const getIPInfo = async (ip) => {
  updateDebugInfo('IP Information', 'loading')
  try {
    const response = await axios.get(`https://ipapi.co/${ip}/json/`, { timeout: 5000 })
    updateDebugInfo('IP Information', 'complete', JSON.stringify(response.data))

    if (response.data.error) {
      throw new Error(`Error getting IP information: ${response.data.reason}`)
    }

    return response.data
  } catch (error) {
    updateDebugInfo('IP Information', 'complete', `Error: ${error.message}`)
    throw error
  }
}

const analyzeDomain = async () => {
  if (!isValidDomain.value) {
    error.value = 'Please enter a valid domain'
    return
  }

  try {
    checkRateLimit()
  } catch (e) {
    error.value = e.message
    return
  }

  isLoading.value = true
  error.value = ''
  debugInfo.value = []
  result.value = null

  const sanitizedDomain = sanitizeInput(domain.value)

  try {
    const ip = await resolveDNS(sanitizedDomain)
    const ipInfo = await getIPInfo(ip)
    const sslStatus = await checkSSL(sanitizedDomain)

    result.value = {
      domain: sanitizedDomain,
      ip: ip,
      country: ipInfo.country_name,
      city: ipInfo.city,
      region: ipInfo.region,
      provider: ipInfo.org,
      ssl_status: sslStatus
    }
  } catch (e) {
    console.error('Error during domain analysis:', e)
    error.value = `Error during domain analysis: ${e.message}`
    updateDebugInfo('Error', 'complete', e.message)
  } finally {
    isLoading.value = false
  }
}
</script>
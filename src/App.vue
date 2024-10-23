<template>
  <div class="min-h-screen bg-gray-100 py-6 flex flex-col justify-center sm:py-12">
    <div class="relative py-3 sm:max-w-4xl sm:mx-auto">
      <div class="absolute inset-0 bg-gradient-to-r from-cyan-400 to-light-blue-500 shadow-lg transform -skew-y-6 sm:skew-y-0 sm:-rotate-6 sm:rounded-3xl"></div>
      <div class="relative px-4 py-10 bg-white shadow-lg sm:rounded-3xl sm:p-20">
        <div class="max-w-3xl mx-auto">
          <h1 class="text-3xl font-semibold mb-6 text-center">Enhanced Domain Analysis</h1>
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
              <div v-show="showDebugInfo" style="overflow: auto; max-height: 300px;" class="mt-2 text-gray-600 p-3 bg-gray-100 rounded-md text-sm">
                <div v-for="(info, index) in debugInfo" :key="index" class="mb-2">
                  <strong>{{ info.action }}:</strong>
                  <span v-if="info.status === 'loading'" class="loading-dots"></span>
                  <pre v-else>{{ info.result }}</pre>
                </div>
              </div>
            </transition>
          </div>
          <div v-if="result" class="mt-6">
            <h2 class="text-2xl font-semibold mb-3 text-center">Analysis Results for {{ result.domain }}</h2>
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
              <div class="bg-gray-50 p-4 rounded-md space-y-2">
                <h3 class="text-lg font-semibold mb-2">Basic Information</h3>
                <p><strong>IP Address:</strong> {{ result.ip || 'Not found' }}</p>
                <p><strong>Country:</strong> {{ result.country || 'Not found' }}</p>
                <p><strong>City:</strong> {{ result.city || 'Not found' }}</p>
                <p><strong>Region:</strong> {{ result.region || 'Not found' }}</p>
                <p><strong>Provider:</strong> {{ result.provider || 'Not found' }}</p>
                <p><strong>SSL Status:</strong> {{ result.ssl_status || 'Not found' }}</p>
                <p><strong>Domain Creation Date:</strong> {{ result.creationDate || 'Not found' }}</p>
              </div>
              <div class="bg-gray-50 p-4 rounded-md space-y-2">
                <h3 class="text-lg font-semibold mb-2">Additional Information</h3>
                <p><strong>Site Availability:</strong> {{ result.siteAvailability || 'Not checked' }}</p>
                <p><strong>Technologies:</strong> {{ result.technologies && result.technologies.length > 0 ? result.technologies.join(', ') : 'None detected' }}</p>
                <p><strong>Mobile Friendly:</strong> {{ result.mobileFriendly || 'Not checked' }}</p>
                <p><strong>Load Time:</strong> {{ result.loadTime ? `${result.loadTime.toFixed(2)}s` : 'Not measured' }}</p>
                <p><strong>Security Headers:</strong> {{ result.securityHeaders && result.securityHeaders.length > 0 ? result.securityHeaders.join(', ') : 'None detected' }}</p>
              </div>
            </div>
            <div v-if="result.screenshot" class="mt-4">
              <h3 class="text-lg font-semibold mb-2">Website Screenshot</h3>
              <div class="relative">
                <div v-if="isScreenshotLoading" class="absolute inset-0 flex items-center justify-center bg-gray-200 rounded-md">
                  <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" class="animate-spin">
                    <path fill="#6a6b70" d="M12 2A10 10 0 1 0 22 12A10 10 0 0 0 12 2Zm0 18a8 8 0 1 1 8-8A8 8 0 0 1 12 20Z" opacity="0.5" />
                    <path fill="#6a6b70" d="M20 12h2A10 10 0 0 0 12 2V4A8 8 0 0 1 20 12Z">
                      <animateTransform attributeName="transform" dur="2s" from="0 12 12" repeatCount="indefinite" to="360 12 12" type="rotate" />
                    </path>
                  </svg>
                </div>
                <img 
                  :src="result.screenshot" 
                  alt="Website Screenshot" 
                  class="w-full rounded-md shadow-md cursor-pointer transition-opacity duration-300" 
                  :class="{ 'opacity-0': isScreenshotLoading, 'opacity-100': !isScreenshotLoading }"
                  @load="onScreenshotLoad" 
                  @error="handleScreenshotError"
                  @click="openFullScreenImage(result.screenshot)"
                />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue'
import axios from 'axios'
import { ChevronDownIcon } from 'lucide-vue-next'

export default {
  components: {
    ChevronDownIcon
  },
  setup() {
    const domain = ref('')
    const result = ref(null)
    const isLoading = ref(false)
    const isScreenshotLoading = ref(true)
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

    const retryRequest = async (func, action, maxRetries = 3) => {
      let retries = 0;
      while (retries < maxRetries) {
        try {
          return await func();
        } catch (error) {
          retries++;
          updateDebugInfo(action, 'loading', `Retry attempt ${retries}/${maxRetries}: ${error.message}`);
          if (retries === maxRetries) {
            updateDebugInfo(action, 'error', `Failed after ${maxRetries} attempts: ${error.message}`);
            throw error;
          }
          await new Promise(resolve => setTimeout(resolve, 2000));
        }
      }
    };

    const checkSSL = async (domain) => {
      updateDebugInfo('Checking SSL', 'loading')
      return retryRequest(async () => {
        const response = await axios.get(`https://${domain}`, { 
          timeout: 10000,
          validateStatus: function (status) {
            return status >= 200 && status < 300 || status === 301 || status === 302
          }
        })
        const result = 'Valid (HTTPS supported)'
        updateDebugInfo('Checking SSL', 'complete', `Result: ${result}\nResponse status: ${response.status}\nResponse headers: ${JSON.stringify(response.headers, null, 2)}`)
        return result
      }, 'Checking SSL');
    }

    const resolveDNS = async (domain) => {
      updateDebugInfo('DNS Resolution', 'loading')
      return retryRequest(async () => {
        const response = await axios.get(`https://dns.google/resolve`, {
          params: { name: domain, type: 'A' },
          timeout: 10000
        })
        updateDebugInfo('DNS Resolution', 'complete', `Response data: ${JSON.stringify(response.data, null, 2)}`)

        if (!response.data.Answer || response.data.Answer.length === 0) {
          throw new Error('Failed to resolve IP address for the domain')
        }

        return response.data.Answer[0].data
      }, 'DNS Resolution');
    }

    const getIPInfo = async (ip) => {
      updateDebugInfo('IP Information', 'loading')
      return retryRequest(async () => {
        const response = await axios.get(`https://ipapi.co/${ip}/json/`, { timeout: 10000 })
        updateDebugInfo('IP Information', 'complete', `Response data: ${JSON.stringify(response.data, null, 2)}`)

        if (response.data.error) {
          throw new Error(`Error getting IP information: ${response.data.reason}`)
        }

        return response.data
      }, 'IP Information');
    }

    const checkSiteAvailability = async (domain) => {
      updateDebugInfo('Site Availability', 'loading')
      return retryRequest(async () => {
        const startTime = performance.now()
        const response = await axios.get(`https://${domain}`, { 
          timeout: 10000,
          validateStatus: function (status) {
            return status >= 200 && status < 300 || status === 301 || status === 302
          }
        })
        const endTime = performance.now()
        const loadTime = (endTime - startTime) / 1000
        const result = 'Available'
        updateDebugInfo('Site Availability', 'complete', `Result: ${result}\nResponse status: ${response.status}\nLoad Time: ${loadTime.toFixed(2)}s`)
        return { status: result, loadTime }
      }, 'Site Availability');
    }

    const detectTechnologies = async (domain) => {
      updateDebugInfo('Technology Detection', 'loading')
      return retryRequest(async () => {
        const response = await axios.get(`https://${domain}`, {
          timeout: 10000,
          headers: {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
          },
          validateStatus: function (status) {
            return status >= 200 && status < 300 || status === 301 || status === 302
          }
        })
        const html = response.data
        const technologies = []

        const techPatterns = {
          'WordPress': [/<link[^>]*wp-content/i, /<script[^>]*wp-includes/i],
          'Shopify': [/<link[^>]*shopify/i, /cdn\.shopify\.com/i],
          'React': [/<div[^>]*id="root"/i, /__REACT_DEVTOOLS_GLOBAL_HOOK__/],
          'Vue.js': [/<div[^>]*id="app"/i, /Vue/],
          'Angular': [/<app-root/i, /ng-version/],
          'Bootstrap': [/<link[^>]*bootstrap/i, /bootstrap\.js/i],
          'jQuery': [/<script[^>]*jquery/i, /jQuery/],
          'Font Awesome': [/<link[^>]*font-awesome/i, /fontawesome/i],
          'Google Analytics': [/google-analytics\.com\/analytics\.js/i, /ga\('create'/],
          'Google Tag Manager': [/googletagmanager\.com\/gtm\.js/i,   /gtm\.start/],
          'Google Fonts': [/fonts\.googleapis\.com/i],
          'Google Maps': [/maps\.googleapis\.com/i],
          'Cloudflare': [/cloudflare/i],
          'Amazon Web Services (AWS)': [/amazonaws\.com/i],
          'Nginx': [/nginx/i],
          'Apache': [/apache/i],
          'PHP': [/X-Powered-By: PHP/i],
          'ASP.NET': [/X-AspNet-Version/i],
          'Laravel': [/laravel/i],
          'Django': [/csrftoken/i],
          'Ruby on Rails': [/rails/i],
          'Node.js': [/node\.js/i, /express/i],
          'Google Cloud': [/googleusercontent\.com/i, /appspot\.com/i],
        }

        for (const [tech, patterns] of Object.entries(techPatterns)) {
          if (patterns.some(pattern => pattern.test(html) || pattern.test(JSON.stringify(response.headers)))) {
            technologies.push(tech)
          }
        }

        const server = response.headers['server']
        if (server) {
          technologies.push(`Server: ${server}`)
        }

        const poweredBy = response.headers['x-powered-by']
        if (poweredBy) {
          technologies.push(`Powered by: ${poweredBy}`)
        }

        updateDebugInfo('Technology Detection', 'complete', `Detected technologies: ${technologies.join(', ')}\nHTML sample: ${html.substring(0, 500)}...\nResponse headers: ${JSON.stringify(response.headers, null, 2)}`)
        return technologies
      }, 'Technology Detection');
    }

    const checkMobileFriendliness = async (domain) => {
      updateDebugInfo('Mobile Friendliness', 'loading')
      return retryRequest(async () => {
        const response = await axios.get(`https://${domain}`, {
          timeout: 10000,
          headers: {
            'User-Agent': 'Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Mobile Safari/537.36'
          },
          validateStatus: function (status) {
            return status >= 200 && status < 300 || status === 301 || status === 302
          }
        })
        const html = response.data
        const viewportTag = html.match(/<meta[^>]*name=["']viewport["'][^>]*content=["'][^"']*width=device-width[^"']*["'][^>]*>/i)
        const mediaQueries = html.match(/@media\s*$$[^)]+$$\s*{[^}]+}/g)
        const flexboxUsage = html.match(/display\s*:\s*flex/i)
        const gridUsage = html.match(/display\s*:\s*grid/i)

        let score = 0
        let reasons = []

        if (viewportTag) {
          score += 2
          reasons.push("Viewport meta tag found")
        } else {
          reasons.push("No viewport meta tag")
        }

        if (mediaQueries && mediaQueries.length > 0) {
          score += 2
          reasons.push(`${mediaQueries.length} media queries found`)
        } else {
          reasons.push("No media queries found")
        }

        if (flexboxUsage || gridUsage) {
          score += 1
          reasons.push("Modern layout techniques (Flexbox/Grid) detected")
        }

        let result
        if (score >= 4) {
          result = "Likely mobile-friendly"
        } else if (score >= 2) {
          result = "Possibly mobile-friendly"
        } else {
          result = "Likely not mobile-friendly"
        }

        updateDebugInfo('Mobile Friendliness', 'complete', `Result: ${result}\nScore: ${score}/5\nReasons: ${reasons.join(", ")}\nHTML sample: ${html.substring(0,500)}...\nResponse headers: ${JSON.stringify(response.headers, null, 2)}`)
        return result
      }, 'Mobile Friendliness');
    }

    const checkSecurityHeaders = async (domain) => {
      updateDebugInfo('Security Headers', 'loading')
      return retryRequest(async () => {
        const response = await axios.get(`https://${domain}`, {
          timeout: 10000,
          validateStatus: function (status) {
            return status >= 200 && status < 300 || status === 301 || status === 302
          }
        })
        const headers = response.headers
        const securityHeaders = []

        if (headers['strict-transport-security']) securityHeaders.push('Strict-Transport-Security')
        if (headers['x-frame-options']) securityHeaders.push('X-Frame-Options')
        if (headers['x-xss-protection']) securityHeaders.push('X-XSS-Protection')
        if (headers['x-content-type-options']) securityHeaders.push('X-Content-Type-Options')
        if (headers['content-security-policy']) securityHeaders.push('Content-Security-Policy')
        if (headers['referrer-policy']) securityHeaders.push('Referrer-Policy')
        if (headers['permissions-policy']) securityHeaders.push('Permissions-Policy')

        updateDebugInfo('Security Headers', 'complete', `Detected headers: ${securityHeaders.join(', ')}\nAll headers: ${JSON.stringify(headers, null, 2)}`)
        return securityHeaders
      }, 'Security Headers');
    }

    const getDomainCreationDate = async (domain) => {
      updateDebugInfo('Domain Creation Date', 'loading')
      return retryRequest(async () => {
        const response = await axios.get(`https://rdap.org/domain/${domain}`, {
          timeout: 10000
        })
        const events = response.data.events
        const creationEvent = events.find(event => event.eventAction === 'registration')
        let creationDate = 'Not found'
        if (creationEvent) {
          const date = new Date(creationEvent.eventDate)
          creationDate = date.toLocaleString('en-US', { 
            year: 'numeric', 
            month: 'long', 
            day: 'numeric',
            hour: '2-digit',
            minute: '2-digit',
            second: '2-digit',
            timeZoneName: 'short'
          })
        }
        updateDebugInfo('Domain Creation Date', 'complete', `Creation Date: ${creationDate}`)
        return creationDate
      }, 'Domain Creation Date');
    }

    const getScreenshot = (domain) => {
      return `https://image.thum.io/get/width/1200/crop/800/noanimate/wait/3/https://${domain}`
    }

    const handleScreenshotError = () => {
      updateDebugInfo('Screenshot', 'complete', 'Failed to load screenshot')
      if (result.value) {
        result.value.screenshot = null
      }
      isScreenshotLoading.value = false
    }

    const onScreenshotLoad = () => {
      isScreenshotLoading.value = false
      updateDebugInfo('Screenshot', 'complete', 'Screenshot loaded successfully')
    }

    const openFullScreenImage = (imageUrl) => {
      window.open(imageUrl, '_blank')
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
      isScreenshotLoading.value = true
      error.value = ''
      debugInfo.value = []
      result.value = null

      const sanitizedDomain = sanitizeInput(domain.value)

      try {
        updateDebugInfo('Analysis Started', 'complete', `Analyzing domain: ${sanitizedDomain}`)

        const analysisResults = await Promise.allSettled([
          resolveDNS(sanitizedDomain),
          checkSSL(sanitizedDomain),
          checkSiteAvailability(sanitizedDomain),
          detectTechnologies(sanitizedDomain),
          checkMobileFriendliness(sanitizedDomain),
          checkSecurityHeaders(sanitizedDomain),
          getDomainCreationDate(sanitizedDomain)
        ])

        const [ip, sslStatus, siteAvailability, technologies, mobileFriendly, securityHeaders, creationDate] = analysisResults.map(result => 
          result.status === 'fulfilled' ? result.value : null
        )

        let ipInfo = null
        if (ip) {
          try {
            ipInfo = await getIPInfo(ip)
          } catch (error) {
            updateDebugInfo('IP Information', 'error', `Failed to get IP info: ${error.message}`)
          }
        }

        const screenshot = getScreenshot(sanitizedDomain)

        result.value = {
          domain: sanitizedDomain,
          ip: ip || 'Not found',
          country: ipInfo?.country_name || 'Not found',
          city: ipInfo?.city || 'Not found',
          region: ipInfo?.region || 'Not found',
          provider: ipInfo?.org || 'Not found',
          ssl_status: sslStatus || 'Not found',
          siteAvailability: siteAvailability?.status || 'Not checked',
          technologies: technologies || [],
          mobileFriendly: mobileFriendly || 'Not checked',
          loadTime: siteAvailability?.loadTime,
          securityHeaders: securityHeaders || [],
          creationDate: creationDate || 'Not found',
          screenshot
        }

        updateDebugInfo('Analysis Completed', 'complete', 'All checks finished')
      } catch (e) {
        console.error('Error during domain analysis:', e)
        error.value = `Error during domain analysis: ${e.message}`
        updateDebugInfo('Error', 'complete', e.message)
      } finally {
        isLoading.value = false
      }
    }

    return {
      domain,
      result,
      isLoading,
      isScreenshotLoading,
      error,
      debugInfo,
      showDebugInfo,
      isValidDomain,
      toggleDebugInfo,
      onEnter,
      onLeave,
      analyzeDomain,
      handleScreenshotError,
      onScreenshotLoad,
      openFullScreenImage
    }
  }
}
</script>

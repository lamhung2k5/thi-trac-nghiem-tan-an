
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { supabase } from './lib/supabase'

const user = ref(null)
const loading = ref(true)
const busy = ref(false)
const errorMessage = ref('')

let subscription = null

onMounted(() => {
  const { data } = supabase.auth.onAuthStateChange(
    (_event, session) => {
      user.value = session?.user ?? null
      loading.value = false
    }
  )

  subscription = data.subscription
})

onUnmounted(() => {
  subscription?.unsubscribe()
})

async function loginWithGoogle() {
  errorMessage.value = ''
  busy.value = true

  const { error } = await supabase.auth.signInWithOAuth({
    provider: 'google',
    options: {
      redirectTo: `${window.location.origin}/`
    }
  })

  if (error) {
    errorMessage.value = error.message
    busy.value = false
  }
}

async function logout() {
  errorMessage.value = ''
  busy.value = true

  const { error } = await supabase.auth.signOut()

  if (error) {
    errorMessage.value = error.message
  }

  busy.value = false
}
</script>

<template>
  <main class="min-vh-100 bg-light d-flex align-items-center justify-content-center p-3">
    <div class="card shadow-sm border-0 w-100" style="max-width: 520px">
      <div class="card-body p-4 p-md-5 text-center">
        <p class="text-primary fw-bold">
          ĐOÀN – HỘI XÃ TÂN AN
        </p>

        <h1 class="h4 fw-bold mb-3">
          Hệ thống thi trắc nghiệm trực tuyến
        </h1>

        <div v-if="loading" class="text-secondary">
          Đang kiểm tra phiên đăng nhập...
        </div>

        <div v-else-if="user">
          <img
            v-if="user.user_metadata?.avatar_url"
            :src="user.user_metadata.avatar_url"
            alt="Ảnh đại diện"
            class="rounded-circle mb-3"
            width="72"
            height="72"
            referrerpolicy="no-referrer"
          />

          <h2 class="h5">
            Xin chào, {{ user.user_metadata?.full_name || 'Thí sinh' }}
          </h2>

          <p class="text-secondary">
            {{ user.email }}
          </p>

          <div class="alert alert-success">
            Đăng nhập Google thành công!
          </div>

          <button
            class="btn btn-outline-danger w-100"
            :disabled="busy"
            @click="logout"
          >
            Đăng xuất
          </button>
        </div>

        <div v-else>
          <p class="text-secondary mb-4">
            Đăng nhập bằng Google để tham gia cuộc thi.
          </p>

          <button
            class="btn btn-primary w-100 py-3"
            :disabled="busy"
            @click="loginWithGoogle"
          >
            {{ busy ? 'Đang chuyển hướng...' : 'Đăng nhập bằng Google' }}
          </button>
        </div>

        <div v-if="errorMessage" class="alert alert-danger mt-3">
          {{ errorMessage }}
        </div>
      </div>
    </div>
  </main>
</template>

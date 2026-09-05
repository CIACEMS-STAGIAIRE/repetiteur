<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue'

import heroImage from '../assets/hero-tutorat-domicile.png'
import logoRepetiteur from '../assets/logo_repetiteur.png'

const navItems = [
  { label: 'Accueil', id: 'accueil' },
  { label: 'Services', id: 'services' },
  { label: 'Méthode', id: 'methode' },
  { label: 'Répétiteurs', id: 'repetiteurs' },
  { label: 'Avis', id: 'avis' },
]

const stats = [
  { value: 12, suffix: '+', label: "années d'expérience" },
  { value: 320, suffix: '+', label: 'familles accompagnées' },
  { value: 95, suffix: '%', label: 'parents satisfaits' },
  { value: 48, suffix: 'h', label: 'pour proposer un profil' },
]

const services = [
  {
    title: 'Primaire et collège',
    text: 'Un suivi régulier pour installer les bases, consolider les acquis et garder le rythme scolaire.',
    accent: 'cyan',
  },
  {
    title: 'Lycée et examens',
    text: 'Des répétiteurs sélectionnés pour les matières scientifiques, littéraires et la préparation aux examens.',
    accent: 'yellow',
  },
  {
    title: 'Suivi personnalisé',
    text: 'Un programme ajusté selon le niveau, les objectifs de la famille et les disponibilités à domicile.',
    accent: 'red',
  },
]

const steps = [
  'Analyse du besoin',
  'Sélection du répétiteur',
  'Mise en relation',
  'Suivi des progrès',
]

const testimonials = [
  {
    quote:
      'Le profil proposé a compris rapidement les difficultés de notre fils. Le suivi est sérieux et les progrès sont visibles.',
    author: 'Parent élève en 3e',
  },
  {
    quote:
      "Nous avions besoin d'un accompagnement en mathématiques avant le bac. Le placement a été rapide et très professionnel.",
    author: 'Famille accompagnée',
  },
]

const displayedStats = ref(stats.map(() => 0))
const hasScrolled = ref(false)
const isMenuOpen = ref(false)
const isSending = ref(false)
const contactStatus = ref<'idle' | 'success' | 'error'>('idle')
const contactMessage = ref('')
const contactForm = ref({
  name: '',
  phone: '',
  location: '',
  level: '',
  subject: '',
  availability: '',
  message: '',
})

const formspreeEndpoint = import.meta.env.VITE_FORMSPREE_ENDPOINT

const updateHeader = () => {
  hasScrolled.value = window.scrollY > 24
}

const syncMobileMenu = () => {
  if (window.innerWidth > 920) {
    isMenuOpen.value = false
  }
}

const closeMobileMenu = () => {
  isMenuOpen.value = false
}

const statValues = computed(() =>
  stats.map((stat, index) => `${Math.round(displayedStats.value[index] ?? 0)}${stat.suffix}`),
)

const sendContactRequest = async () => {
  contactStatus.value = 'idle'
  contactMessage.value = ''

  if (!formspreeEndpoint) {
    contactStatus.value = 'error'
    contactMessage.value = 'La configuration du formulaire est incomplète.'
    return
  }

  isSending.value = true

  try {
    const response = await fetch(formspreeEndpoint, {
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        _subject: `Nouvelle demande de répétiteur - ${contactForm.value.name}`,
        'Nom complet': contactForm.value.name,
        Téléphone: contactForm.value.phone,
        'Commune / quartier': contactForm.value.location,
        'Niveau scolaire': contactForm.value.level,
        'Matière principale': contactForm.value.subject,
        'Disponibilité souhaitée': contactForm.value.availability,
        'Besoin principal': contactForm.value.message,
      }),
    })

    if (!response.ok) {
      const data = (await response.json().catch(() => null)) as {
        errors?: Array<{ message?: string }>
      } | null
      const formspreeError = data?.errors
        ?.map((error) => error.message)
        .filter(Boolean)
        .join(' ')

      throw new Error(formspreeError || `Erreur ${response.status}`)
    }

    contactStatus.value = 'success'
    contactMessage.value = 'Votre demande a bien été envoyée. Nous vous recontacterons rapidement.'
    contactForm.value = {
      name: '',
      phone: '',
      location: '',
      level: '',
      subject: '',
      availability: '',
      message: '',
    }
  } catch (error) {
    const formError = error instanceof Error ? error.message : ''

    contactStatus.value = 'error'
    contactMessage.value = formError
      ? `L'envoi a échoué : ${formError}`
      : "L'envoi a échoué. Vérifiez votre connexion puis réessayez."
  } finally {
    isSending.value = false
  }
}

onMounted(() => {
  updateHeader()
  window.addEventListener('scroll', updateHeader, { passive: true })
  window.addEventListener('resize', syncMobileMenu)

  const duration = 1500
  const start = performance.now()

  const animate = (now: number) => {
    const progress = Math.min((now - start) / duration, 1)
    const eased = 1 - Math.pow(1 - progress, 3)

    displayedStats.value = stats.map((stat) => stat.value * eased)

    if (progress < 1) {
      requestAnimationFrame(animate)
    }
  }

  requestAnimationFrame(animate)
})

onUnmounted(() => {
  window.removeEventListener('scroll', updateHeader)
  window.removeEventListener('resize', syncMobileMenu)
})
</script>

<template>
  <main class="home-page">
    <header class="site-header" :class="{ 'site-header--transparent': hasScrolled }">
      <div class="header-inner">
        <a class="brand" href="#accueil" aria-label="Répétiteur Plus - accueil">
          <span class="brand-surface">
            <img :src="logoRepetiteur" alt="Répétiteur Plus" />
          </span>
        </a>

        <nav class="main-nav" aria-label="Navigation principale">
          <a v-for="item in navItems" :key="item.id" :href="`#${item.id}`" @click="closeMobileMenu">
            {{ item.label }}
          </a>
        </nav>

        <div class="header-actions">
          <a class="contact-button" href="#contact" @click="closeMobileMenu">Contacter</a>
          <button
            class="menu-toggle"
            :class="{ 'menu-toggle--open': isMenuOpen }"
            type="button"
            :aria-expanded="isMenuOpen"
            aria-controls="mobile-menu"
            :aria-label="isMenuOpen ? 'Fermer le menu' : 'Ouvrir le menu'"
            @click="isMenuOpen = !isMenuOpen"
          >
            <span></span>
            <span></span>
            <span></span>
          </button>
        </div>

        <nav
          id="mobile-menu"
          class="mobile-nav"
          :class="{ 'mobile-nav--open': isMenuOpen }"
          aria-label="Navigation mobile"
        >
          <a
            v-for="item in navItems"
            :key="`mobile-${item.id}`"
            :href="`#${item.id}`"
            @click="closeMobileMenu"
          >
            {{ item.label }}
          </a>
        </nav>
      </div>
    </header>

    <section id="accueil" class="hero" :style="{ backgroundImage: `url(${heroImage})` }">
      <div class="hero-overlay">
        <div class="hero-content">
          <p class="eyebrow">Placement de répétiteurs à domicile</p>
          <h1>Le bon répétiteur, au bon moment, pour chaque élève.</h1>
          <p class="hero-text">
            Nous identifions le besoin, sélectionnons un profil fiable et organisons un
            accompagnement à domicile adapté au rythme de votre enfant.
          </p>

          <div class="hero-actions">
            <a class="primary-action" href="#contact">Demander un répétiteur</a>
            <a class="secondary-action" href="#methode">Voir la méthode</a>
          </div>

          <div class="trust-line">
            <span></span>
            Profils vérifiés, suivi régulier et mise en relation rapide.
          </div>
        </div>
      </div>
    </section>

    <section class="stats-section" aria-label="Statistiques">
      <div class="stats-grid">
        <article v-for="(stat, index) in stats" :key="stat.label" class="stat-card">
          <strong>{{ statValues[index] }}</strong>
          <span>{{ stat.label }}</span>
        </article>
      </div>
    </section>

    <section id="services" class="section services-section">
      <div class="section-heading">
        <p class="eyebrow">Nos accompagnements</p>
        <h2>Des solutions claires pour les familles exigeantes.</h2>
      </div>

      <div class="services-grid">
        <article
          v-for="service in services"
          :key="service.title"
          class="service-card"
          :class="`service-card--${service.accent}`"
        >
          <div class="service-mark" aria-hidden="true"></div>
          <h3>{{ service.title }}</h3>
          <p>{{ service.text }}</p>
        </article>
      </div>
    </section>

    <section id="methode" class="section method-section">
      <div class="method-copy">
        <p class="eyebrow">Méthode</p>
        <h2>Un placement simple, encadré et mesurable.</h2>
        <p>
          Chaque demande est traitée comme un dossier d'accompagnement: niveau scolaire, matières
          prioritaires, disponibilités, personnalité de l'élève et objectifs de la famille.
        </p>
      </div>

      <div class="timeline">
        <article v-for="(step, index) in steps" :key="step" class="timeline-item">
          <span>{{ String(index + 1).padStart(2, '0') }}</span>
          <h3>{{ step }}</h3>
        </article>
      </div>
    </section>

    <section id="repetiteurs" class="section profile-section">
      <div class="profile-panel">
        <p class="eyebrow">Sélection</p>
        <h2>Des répétiteurs choisis pour leur niveau et leur régularité.</h2>
        <p>
          La structure privilégie des profils pédagogues, ponctuels et capables de rendre compte des
          progrès après chaque période de suivi.
        </p>
      </div>

      <div class="quality-list">
        <span>Vérification du niveau</span>
        <span>Expérience par matière</span>
        <span>Disponibilités confirmées</span>
        <span>Suivi avec les parents</span>
      </div>
    </section>

    <section id="avis" class="section testimonials-section">
      <div class="section-heading">
        <p class="eyebrow">Avis parents</p>
        <h2>La confiance se construit dans le suivi.</h2>
      </div>

      <div class="testimonials-grid">
        <article v-for="testimonial in testimonials" :key="testimonial.author" class="quote-card">
          <p>"{{ testimonial.quote }}"</p>
          <strong>{{ testimonial.author }}</strong>
        </article>
      </div>
    </section>

    <section id="contact" class="contact-section">
      <div>
        <p class="eyebrow">Contact</p>
        <h2>Parlez-nous du niveau et des besoins de votre enfant.</h2>
        <p>
          Une équipe vous recontacte pour proposer un accompagnement adapté et un répétiteur
          disponible.
        </p>

        <div class="contact-direct" aria-label="Coordonnées directes">
          <a href="tel:+2250758034072">
            <span class="contact-icon" aria-hidden="true">
              <svg viewBox="0 0 24 24" role="img">
                <path
                  d="M22 16.92v3a2 2 0 0 1-2.18 2 19.79 19.79 0 0 1-8.63-3.07 19.5 19.5 0 0 1-6-6A19.79 19.79 0 0 1 2.12 4.18 2 2 0 0 1 4.11 2h3a2 2 0 0 1 2 1.72c.13.96.35 1.9.66 2.81a2 2 0 0 1-.45 2.11L8.05 9.91a16 16 0 0 0 6.04 6.04l1.27-1.27a2 2 0 0 1 2.11-.45c.91.31 1.85.53 2.81.66A2 2 0 0 1 22 16.92Z"
                />
              </svg>
            </span>
            <strong>+225 07 58 03 40 72</strong>
          </a>
          <a href="mailto:biogneachille@gmail.com">
            <span class="contact-icon" aria-hidden="true">
              <svg viewBox="0 0 24 24" role="img">
                <path d="M4 4h16a2 2 0 0 1 2 2v12a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2Z" />
                <path d="m22 7-10 6L2 7" />
              </svg>
            </span>
            <strong>biogneachille@gmail.com</strong>
          </a>
          <a
            class="whatsapp-button"
            href="https://wa.me/2250758034072?text=Bonjour%2C%20je%20souhaite%20avoir%20des%20informations%20sur%20un%20r%C3%A9p%C3%A9titeur%20%C3%A0%20domicile."
            target="_blank"
            rel="noopener noreferrer"
          >
            <span class="contact-icon" aria-hidden="true">
              <svg viewBox="0 0 24 24" role="img">
                <path
                  d="M20.52 3.48A11.87 11.87 0 0 0 12.08 0 11.98 11.98 0 0 0 1.9 18.33L.16 24l5.82-1.68A11.94 11.94 0 0 0 12.08 24 11.99 11.99 0 0 0 24 11.92a11.86 11.86 0 0 0-3.48-8.44ZM12.08 21.9a9.9 9.9 0 0 1-5.05-1.38l-.36-.21-3.45 1 .99-3.35-.23-.38A9.88 9.88 0 1 1 12.08 21.9Zm5.43-7.41c-.3-.15-1.76-.87-2.03-.97-.27-.1-.47-.15-.67.15-.2.3-.77.97-.94 1.17-.17.2-.35.22-.65.08-.3-.15-1.26-.46-2.4-1.48-.89-.79-1.49-1.77-1.66-2.07-.17-.3-.02-.46.13-.61.13-.13.3-.35.45-.52.15-.18.2-.3.3-.5.1-.2.05-.37-.03-.52-.07-.15-.67-1.61-.92-2.21-.24-.58-.49-.5-.67-.51h-.57c-.2 0-.52.07-.79.37-.27.3-1.04 1.02-1.04 2.48s1.07 2.88 1.22 3.08c.15.2 2.1 3.2 5.08 4.49.71.31 1.26.49 1.69.63.71.23 1.36.2 1.88.12.57-.09 1.76-.72 2.01-1.42.25-.7.25-1.29.17-1.42-.08-.13-.27-.2-.57-.35Z"
                />
              </svg>
            </span>
            <strong>Écrire sur WhatsApp</strong>
          </a>
        </div>
      </div>

      <form
        class="contact-form"
        aria-label="Formulaire de contact"
        @submit.prevent="sendContactRequest"
      >
        <input
          v-model="contactForm.name"
          type="text"
          name="name"
          placeholder="Nom et prénoms"
          aria-label="Nom et prénoms"
          required
        />
        <input
          v-model="contactForm.phone"
          type="tel"
          name="phone"
          placeholder="Téléphone"
          aria-label="Téléphone"
          required
        />
        <input
          v-model="contactForm.location"
          type="text"
          name="location"
          placeholder="Commune / quartier"
          aria-label="Commune ou quartier"
          required
        />
        <select v-model="contactForm.level" name="level" aria-label="Niveau scolaire" required>
          <option value="" disabled>Niveau scolaire</option>
          <option>Primaire</option>
          <option>Collège</option>
          <option>Lycée</option>
          <option>Préparation examen</option>
        </select>
        <select
          v-model="contactForm.subject"
          name="subject"
          aria-label="Matière principale"
          required
        >
          <option value="" disabled>Matière principale</option>
          <option>Mathématiques</option>
          <option>Français</option>
          <option>Anglais</option>
          <option>Physique-Chimie</option>
          <option>SVT</option>
          <option>Aide aux devoirs</option>
          <option>Plusieurs matières</option>
        </select>
        <select
          v-model="contactForm.availability"
          name="availability"
          aria-label="Disponibilité souhaitée"
          required
        >
          <option value="" disabled>Disponibilité souhaitée</option>
          <option>Matin</option>
          <option>Après-midi</option>
          <option>Soir en semaine</option>
          <option>Week-end</option>
          <option>À définir ensemble</option>
        </select>
        <textarea
          v-model="contactForm.message"
          name="message"
          placeholder="Besoin principal"
          aria-label="Besoin principal"
          required
        ></textarea>
        <button type="submit" :disabled="isSending">
          {{ isSending ? 'Envoi en cours...' : 'Envoyer la demande' }}
        </button>
        <p v-if="contactMessage" class="form-status" :class="`form-status--${contactStatus}`">
          {{ contactMessage }}
        </p>
      </form>
    </section>

    <footer class="site-footer">
      <div class="footer-main">
        <div class="footer-brand">
          <img :src="logoRepetiteur" alt="Répétiteur Plus" />
          <p>Placement de répétiteurs à domicile pour un suivi scolaire fiable et personnalisé.</p>
        </div>

        <address class="footer-contact">
          <h2>Contact</h2>
          <a href="tel:+2250758034072">+225 07 58 03 40 72</a>
          <a href="mailto:biogneachille@gmail.com">biogneachille@gmail.com</a>
          <span>Abidjan, Côte d'Ivoire</span>
        </address>

        <nav class="footer-links" aria-label="Raccourcis footer">
          <h2>Menu</h2>
          <a v-for="item in navItems" :key="`footer-${item.id}`" :href="`#${item.id}`">
            {{ item.label }}
          </a>
        </nav>
      </div>

      <div class="footer-bottom">
        <p>© 2026 Répétiteur Plus. Tous droits réservés.</p>
        <p>
          Développé par
          <a href="https://ciacems.net" target="_blank" rel="noopener noreferrer">CIACEMS</a>
        </p>
      </div>
    </footer>
  </main>
</template>

<style scoped>
.home-page {
  min-height: 100vh;
  overflow-x: hidden;
}

.site-header {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  z-index: 20;
  width: 100%;
  min-height: 92px;
  background: rgba(255, 255, 255, 0.96);
  border-bottom: 1px solid rgba(221, 229, 242, 0.95);
  box-shadow: 0 16px 38px rgba(14, 23, 48, 0.08);
  backdrop-filter: blur(18px);
  transition:
    background 220ms ease,
    border-color 220ms ease,
    box-shadow 220ms ease;
}

.site-header--transparent {
  background: rgba(255, 255, 255, 0.08);
  border-bottom-color: rgba(255, 255, 255, 0.16);
  box-shadow: none;
}

.header-inner {
  display: grid;
  grid-template-columns: auto 1fr auto;
  align-items: center;
  gap: 24px;
  width: min(1180px, calc(100% - 32px));
  min-height: 92px;
  margin: 0 auto;
}

.brand {
  display: inline-flex;
  align-items: center;
}

.brand-surface {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: clamp(172px, 18vw, 236px);
  height: 68px;
  padding: 8px 16px;
  border: 1px solid rgba(1, 195, 247, 0.22);
  border-radius: var(--radius);
  background: rgba(255, 255, 255, 0.98);
  box-shadow: 0 14px 32px rgba(0, 42, 144, 0.18);
}

.brand img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  object-position: center;
}

.main-nav {
  display: flex;
  justify-content: center;
  gap: clamp(14px, 2vw, 30px);
  color: var(--color-muted);
  font-size: 0.94rem;
  font-weight: 700;
}

.site-header--transparent .main-nav {
  color: var(--color-cyan);
  text-shadow: 0 2px 12px rgba(0, 22, 83, 0.72);
}

.header-actions {
  display: inline-flex;
  align-items: center;
  gap: 12px;
  justify-self: end;
}

.menu-toggle {
  display: none;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  gap: 5px;
  width: 46px;
  height: 46px;
  padding: 0;
  color: var(--color-primary);
  border: 1px solid rgba(1, 195, 247, 0.28);
  border-radius: var(--radius);
  background: rgba(255, 255, 255, 0.96);
  box-shadow: 0 12px 24px rgba(0, 42, 144, 0.12);
  cursor: pointer;
}

.menu-toggle span {
  width: 20px;
  height: 2px;
  border-radius: 999px;
  background: currentColor;
  transition:
    opacity 180ms ease,
    transform 180ms ease;
}

.menu-toggle--open span:first-child {
  transform: translateY(7px) rotate(45deg);
}

.menu-toggle--open span:nth-child(2) {
  opacity: 0;
}

.menu-toggle--open span:last-child {
  transform: translateY(-7px) rotate(-45deg);
}

.mobile-nav {
  display: none;
}

.main-nav a {
  position: relative;
  padding: 10px 0;
}

.main-nav a::after {
  position: absolute;
  right: 0;
  bottom: 2px;
  left: 0;
  height: 3px;
  content: '';
  background: var(--color-cyan);
  transform: scaleX(0);
  transform-origin: center;
  transition: transform 180ms ease;
}

.main-nav a:hover::after {
  transform: scaleX(1);
}

.contact-button,
.primary-action,
.secondary-action,
.contact-form button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-height: 48px;
  padding: 0 22px;
  border-radius: var(--radius);
  font-weight: 800;
  transition:
    transform 180ms ease,
    box-shadow 180ms ease,
    background 180ms ease;
}

.contact-button,
.primary-action,
.contact-form button {
  color: var(--color-white);
  background: var(--color-red);
  box-shadow: 0 14px 26px rgba(245, 2, 1, 0.22);
}

.contact-button:hover,
.primary-action:hover,
.contact-form button:hover {
  transform: translateY(-2px);
  box-shadow: 0 18px 32px rgba(245, 2, 1, 0.28);
}

.hero {
  min-height: clamp(650px, 78vh, 820px);
  margin-top: 0;
  background-repeat: no-repeat;
  background-position: center;
  background-size: cover;
}

.hero-overlay {
  display: flex;
  align-items: center;
  min-height: inherit;
  padding: 130px 0 68px;
  background:
    linear-gradient(
      90deg,
      rgba(0, 22, 83, 0.88) 0%,
      rgba(0, 42, 144, 0.72) 38%,
      rgba(0, 42, 144, 0.18) 72%
    ),
    linear-gradient(0deg, rgba(0, 22, 83, 0.28), rgba(0, 22, 83, 0));
}

.hero-content {
  width: min(1180px, calc(100% - 32px));
  margin: 0 auto;
  color: var(--color-white);
}

.eyebrow {
  margin: 0 0 14px;
  color: var(--color-cyan);
  font-size: 0.78rem;
  font-weight: 900;
  letter-spacing: 0;
  text-transform: uppercase;
}

.hero h1 {
  max-width: 650px;
  margin: 0;
  font-size: clamp(2.7rem, 7vw, 5.8rem);
  line-height: 0.98;
  letter-spacing: 0;
}

.hero-text {
  max-width: 610px;
  margin: 26px 0 0;
  color: rgba(255, 255, 255, 0.86);
  font-size: clamp(1.05rem, 2vw, 1.25rem);
  line-height: 1.7;
}

.hero-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-top: 34px;
}

.secondary-action {
  color: var(--color-white);
  border: 1px solid rgba(255, 255, 255, 0.4);
  background: rgba(255, 255, 255, 0.12);
}

.secondary-action:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-2px);
}

.trust-line {
  display: flex;
  align-items: center;
  gap: 12px;
  max-width: 560px;
  margin-top: 32px;
  color: rgba(255, 255, 255, 0.82);
  font-weight: 700;
}

.trust-line span {
  width: 46px;
  height: 5px;
  border-radius: 999px;
  background: var(--color-yellow);
}

.stats-section {
  position: relative;
  z-index: 3;
  width: min(1080px, calc(100% - 32px));
  margin: -46px auto 0;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  overflow: hidden;
  background: var(--color-white);
  border: 1px solid var(--color-line);
  border-radius: var(--radius);
  box-shadow: var(--shadow-soft);
}

.stat-card {
  min-height: 142px;
  padding: 28px 22px;
  border-right: 1px solid var(--color-line);
}

.stat-card:last-child {
  border-right: 0;
}

.stat-card strong {
  display: block;
  color: var(--color-primary);
  font-size: clamp(2rem, 5vw, 3.15rem);
  line-height: 1;
}

.stat-card span {
  display: block;
  margin-top: 12px;
  color: var(--color-muted);
  font-weight: 800;
}

.section {
  width: min(1180px, calc(100% - 32px));
  margin: 0 auto;
  padding: 96px 0;
}

.section-heading {
  max-width: 710px;
  margin-bottom: 34px;
}

.section h2,
.contact-section h2 {
  margin: 0;
  color: var(--color-primary-dark);
  font-size: clamp(2rem, 4vw, 3.35rem);
  line-height: 1.05;
}

.services-grid,
.testimonials-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 18px;
}

.service-card,
.quote-card {
  min-height: 250px;
  padding: 28px;
  border: 1px solid var(--color-line);
  border-radius: var(--radius);
  background: var(--color-white);
  box-shadow: 0 12px 30px rgba(14, 23, 48, 0.06);
}

.service-mark {
  width: 48px;
  height: 8px;
  margin-bottom: 28px;
  border-radius: 999px;
  background: var(--color-cyan);
}

.service-card--yellow .service-mark {
  background: var(--color-yellow);
}

.service-card--red .service-mark {
  background: var(--color-red);
}

.service-card h3,
.timeline-item h3 {
  margin: 0 0 14px;
  color: var(--color-primary);
  font-size: 1.25rem;
}

.service-card p,
.method-copy p,
.profile-panel p,
.quote-card p,
.contact-section p {
  margin: 0;
  color: var(--color-muted);
  line-height: 1.75;
}

.method-section {
  display: grid;
  grid-template-columns: 0.9fr 1.1fr;
  gap: 44px;
  align-items: center;
  border-top: 1px solid var(--color-line);
  border-bottom: 1px solid var(--color-line);
}

.method-copy p {
  margin-top: 20px;
  font-size: 1.05rem;
}

.timeline {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 14px;
}

.timeline-item {
  min-height: 148px;
  padding: 24px;
  border-radius: var(--radius);
  background: var(--color-soft);
}

.timeline-item span {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 42px;
  height: 42px;
  margin-bottom: 22px;
  color: var(--color-primary-dark);
  background: var(--color-yellow);
  border-radius: 50%;
  font-weight: 900;
}

.profile-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 36px;
  align-items: stretch;
}

.profile-panel {
  padding: 44px;
  color: var(--color-white);
  border-radius: var(--radius);
  background: var(--color-primary);
}

.profile-panel .eyebrow,
.profile-panel h2,
.profile-panel p {
  color: var(--color-white);
}

.profile-panel p {
  margin-top: 20px;
  opacity: 0.84;
}

.quality-list {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 14px;
}

.quality-list span {
  display: flex;
  align-items: end;
  min-height: 136px;
  padding: 22px;
  color: var(--color-primary-dark);
  border: 1px solid var(--color-line);
  border-left: 7px solid var(--color-cyan);
  border-radius: var(--radius);
  background: var(--color-white);
  font-size: 1.05rem;
  font-weight: 900;
}

.quality-list span:nth-child(2) {
  border-left-color: var(--color-yellow);
}

.quality-list span:nth-child(3) {
  border-left-color: var(--color-red);
}

.testimonials-section {
  padding-top: 18px;
}

.testimonials-grid {
  grid-template-columns: repeat(2, 1fr);
}

.quote-card {
  min-height: 220px;
  background: linear-gradient(180deg, var(--color-white), #f9fbff);
}

.quote-card p {
  color: var(--color-ink);
  font-size: 1.05rem;
}

.quote-card strong {
  display: block;
  margin-top: 22px;
  color: var(--color-primary);
}

.contact-section {
  display: grid;
  grid-template-columns: 0.9fr 1.1fr;
  gap: 42px;
  align-items: stretch;
  width: min(1180px, calc(100% - 32px));
  margin: 0 auto 88px;
  padding: 52px;
  color: var(--color-white);
  border-radius: var(--radius);
  background:
    linear-gradient(135deg, rgba(0, 42, 144, 0.96), rgba(0, 22, 83, 0.98)), var(--color-primary);
}

.contact-section h2,
.contact-section p {
  color: var(--color-white);
}

.contact-section p {
  margin-top: 18px;
  opacity: 0.82;
}

.contact-section > div {
  display: flex;
  flex-direction: column;
}

.contact-direct {
  display: flex;
  flex: 1;
  flex-direction: column;
  gap: 4px;
  max-width: 440px;
  margin-top: 34px;
}

.contact-direct a {
  display: flex;
  align-items: center;
  gap: 14px;
  min-height: 44px;
  padding: 6px 0;
  color: var(--color-white);
  font-size: 1.05rem;
  font-weight: 900;
  transition:
    color 180ms ease,
    transform 180ms ease;
}

.contact-direct a:hover {
  color: var(--color-cyan);
  transform: translateY(-2px);
}

.contact-direct .whatsapp-button {
  justify-content: center;
  min-height: 52px;
  margin-top: auto;
  padding: 0 18px;
  color: var(--color-white);
  border-radius: var(--radius);
  background: #25d366;
  box-shadow: 0 14px 28px rgba(37, 211, 102, 0.2);
}

.contact-direct .whatsapp-button:hover {
  color: var(--color-white);
  background: #1fb85a;
}

.whatsapp-button .contact-icon {
  width: 24px;
  height: 24px;
}

.whatsapp-button .contact-icon svg {
  fill: currentColor;
  stroke: none;
}

.contact-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  flex: 0 0 auto;
  width: 28px;
  height: 28px;
  color: currentColor;
}

.contact-icon svg {
  width: 20px;
  height: 20px;
  fill: none;
  stroke: currentColor;
  stroke-linecap: round;
  stroke-linejoin: round;
  stroke-width: 2;
}

.contact-direct strong {
  overflow-wrap: anywhere;
}

.contact-form {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: repeat(3, 50px) 1fr 52px;
  gap: 14px;
  height: 100%;
  min-height: 430px;
}

.contact-form input,
.contact-form select,
.contact-form textarea {
  width: 100%;
  min-height: 50px;
  padding: 0 16px;
  color: var(--color-ink);
  border: 1px solid rgba(255, 255, 255, 0.22);
  border-radius: var(--radius);
  background: var(--color-white);
  outline: none;
}

.contact-form textarea {
  grid-column: 1 / -1;
  min-height: 100%;
  padding-top: 14px;
  resize: vertical;
}

.contact-form button {
  grid-column: 1 / -1;
  height: 52px;
  border: 0;
  cursor: pointer;
}

.contact-form button:disabled {
  cursor: wait;
  opacity: 0.72;
  transform: none;
}

.form-status {
  grid-column: 1 / -1;
  margin: 2px 0 0;
  padding: 12px 14px;
  border-radius: var(--radius);
  font-weight: 800;
  line-height: 1.45;
}

.form-status--success {
  color: var(--color-primary-dark);
  background: rgba(245, 199, 89, 0.95);
}

.form-status--error {
  color: var(--color-white);
  background: rgba(245, 2, 1, 0.86);
}

.site-footer {
  color: rgba(255, 255, 255, 0.78);
  background: var(--color-primary-dark);
}

.footer-main {
  display: grid;
  grid-template-columns: 1.35fr 1fr 0.8fr;
  gap: 42px;
  width: min(1180px, calc(100% - 32px));
  margin: 0 auto;
  padding: 56px 0 42px;
}

.footer-brand img {
  width: 190px;
  height: 62px;
  object-fit: contain;
  object-position: left center;
  padding: 8px 12px;
  border-radius: var(--radius);
  background: var(--color-white);
}

.footer-brand p {
  max-width: 420px;
  margin: 18px 0 0;
  line-height: 1.7;
}

.footer-contact,
.footer-links {
  display: grid;
  align-content: start;
  gap: 12px;
  font-style: normal;
}

.footer-contact h2,
.footer-links h2 {
  margin: 0;
  color: var(--color-white);
  font-size: 1.05rem;
}

.footer-contact a,
.footer-contact span,
.footer-links a {
  color: rgba(255, 255, 255, 0.78);
  line-height: 1.5;
}

.footer-contact a:hover,
.footer-links a:hover,
.footer-bottom a:hover {
  color: var(--color-cyan);
}

.footer-bottom {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
  padding: 28px max(16px, calc((100% - 1180px) / 2)) 30px;
  color: #52658d;
  background: var(--color-white);
  border-top: 1px solid var(--color-line);
}

.footer-bottom p {
  margin: 0;
}

.footer-bottom a {
  color: var(--color-primary);
  font-weight: 900;
}

@media (max-width: 920px) {
  .site-header {
    min-height: 82px;
    background: rgba(255, 255, 255, 0.96);
  }

  .site-header--transparent {
    background: rgba(255, 255, 255, 0.08);
  }

  .header-inner {
    position: relative;
    grid-template-columns: auto 1fr;
    min-height: 82px;
  }

  .main-nav {
    display: none;
  }

  .menu-toggle {
    display: inline-flex;
  }

  .mobile-nav {
    position: absolute;
    top: calc(100% + 10px);
    right: 0;
    left: 0;
    display: grid;
    gap: 4px;
    padding: 10px;
    border: 1px solid rgba(221, 229, 242, 0.95);
    border-radius: var(--radius);
    background: rgba(255, 255, 255, 0.98);
    box-shadow: 0 20px 45px rgba(0, 22, 83, 0.2);
    opacity: 0;
    pointer-events: none;
    transform: translateY(-8px);
    transition:
      opacity 180ms ease,
      transform 180ms ease;
  }

  .mobile-nav--open {
    opacity: 1;
    pointer-events: auto;
    transform: translateY(0);
  }

  .mobile-nav a {
    padding: 13px 14px;
    color: var(--color-primary-dark);
    border-radius: var(--radius);
    font-weight: 900;
  }

  .mobile-nav a:hover {
    color: var(--color-primary);
    background: var(--color-soft);
  }

  .hero {
    margin-top: 0;
  }

  .hero-overlay {
    padding-top: 124px;
    background:
      linear-gradient(
        90deg,
        rgba(0, 22, 83, 0.92) 0%,
        rgba(0, 42, 144, 0.7) 58%,
        rgba(0, 42, 144, 0.18) 100%
      ),
      linear-gradient(0deg, rgba(0, 22, 83, 0.35), rgba(0, 22, 83, 0));
  }

  .stats-grid,
  .method-section,
  .profile-section,
  .testimonials-grid,
  .contact-section {
    grid-template-columns: 1fr;
  }

  .footer-main {
    grid-template-columns: 1fr 1fr;
  }

  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .stat-card {
    border-bottom: 1px solid var(--color-line);
  }

  .stat-card:nth-child(odd) {
    border-right: 1px solid var(--color-line);
  }

  .stat-card:nth-child(even) {
    border-right: 0;
  }

  .stat-card:nth-last-child(-n + 2) {
    border-bottom: 0;
  }

  .stat-card:last-child {
    border-right: 0;
    border-bottom: 0;
  }

  .services-grid {
    grid-template-columns: 1fr;
  }

  .contact-section {
    padding: 34px 22px;
  }

  .contact-direct .whatsapp-button {
    margin-top: 18px;
  }
}

@media (max-width: 620px) {
  .header-inner {
    width: min(100% - 20px, 1180px);
    gap: 12px;
  }

  .brand-surface {
    width: 138px;
    height: 56px;
    padding: 7px 10px;
  }

  .contact-button {
    min-height: 42px;
    padding: 0 12px;
    font-size: 0.9rem;
  }

  .menu-toggle {
    width: 42px;
    height: 42px;
  }

  .hero {
    margin-top: 82px;
    min-height: calc(100svh - 82px);
    background-color: var(--color-primary-dark);
    background-position: center top;
    background-size: contain;
  }

  .hero-overlay {
    align-items: center;
    min-height: calc(100svh - 82px);
    padding: 64px 0 42px;
    background:
      linear-gradient(
        180deg,
        rgba(0, 22, 83, 0.42) 0%,
        rgba(0, 42, 144, 0.74) 34%,
        rgba(0, 22, 83, 0.98) 78%
      ),
      linear-gradient(90deg, rgba(0, 22, 83, 0.88), rgba(0, 42, 144, 0.2));
  }

  .stat-card {
    min-height: 132px;
    padding: 24px 20px;
  }

  .hero-actions,
  .trust-line {
    align-items: stretch;
    flex-direction: column;
  }

  .primary-action,
  .secondary-action {
    width: 100%;
  }

  .section {
    padding: 72px 0;
  }

  .timeline,
  .quality-list,
  .contact-form {
    grid-template-columns: 1fr;
  }

  .contact-form {
    grid-template-rows: none;
    height: auto;
    min-height: 0;
  }

  .contact-form textarea {
    min-height: 128px;
  }

  .contact-form textarea,
  .contact-form button,
  .form-status {
    grid-column: auto;
  }

  .footer-main {
    grid-template-columns: 1fr;
    gap: 30px;
    padding: 44px 0 34px;
  }

  .footer-bottom {
    align-items: flex-start;
    flex-direction: column;
    gap: 10px;
  }
}
</style>

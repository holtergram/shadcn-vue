<script setup lang="ts">
import { cn } from '@/lib/utils'
import { Button } from '@/registry/default/ui/button'
import { Command, CommandEmpty, CommandGroup, CommandInput, CommandItem, CommandList, CommandSeparator } from '@/registry/default/ui/command'
import { Dialog, DialogContent } from '@/registry/default/ui/dialog'
import { Toaster as DefaultToaster } from '@/registry/default/ui/toast'
import { Toaster as NewYorkSonner } from '@/registry/new-york/ui/sonner'
import { Toaster as NewYorkToaster } from '@/registry/new-york/ui/toast'

import { TooltipProvider } from '@/registry/new-york/ui/tooltip'
import { useConfigStore } from '@/stores/config'
import { useMagicKeys, useToggle } from '@vueuse/core'

import { Content, useData, useRoute, useRouter } from 'vitepress'
import { onMounted, ref, watch } from 'vue'
import MoonIcon from '~icons/lucide/moon'
import SunIcon from '~icons/lucide/sun'
import Circle from '~icons/radix-icons/circle'
import File from '~icons/radix-icons/file'
import GithubLogoIcon from '~icons/radix-icons/github-logo'
import CodeConfigCustomizer from '../components/CodeConfigCustomizer.vue'
import Kbd from '../components/Kbd.vue'
import Logo from '../components/Logo.vue'

import MobileNav from '../components/MobileNav.vue'
import ThemePopover from '../components/ThemePopover.vue'
import { docsConfig, type NavItem } from '../config/docs'

const { radius, theme } = useConfigStore()
// Whenever the component is mounted, update the document class list
onMounted(() => {
  document.documentElement.style.setProperty('--radius', `${radius.value}rem`)
  document.documentElement.classList.add(`theme-${theme.value}`)
})

const { frontmatter, isDark } = useData()

const $route = useRoute()
const $router = useRouter()
const toggleDark = useToggle(isDark)

const links = [
  {
    name: 'GitHub',
    href: 'https://github.com/unovue/shadcn-vue',
    icon: GithubLogoIcon,
  },
]

const isOpen = ref(false)
const { Meta_K, Ctrl_K } = useMagicKeys({
  passive: false,
  onEventFired(e) {
    if (e.key === 'k' && (e.metaKey || e.ctrlKey))
      e.preventDefault()
  },
})

watch([Meta_K, Ctrl_K], (v) => {
  if (v[0] || v[1])
    isOpen.value = true
})

function handleSelectLink(item: NavItem) {
  if (item.external)
    window.open(item.href, '_blank')
  else
    $router.go(item.href)

  isOpen.value = false
}

watch(() => $route.path, (n) => {
  if (document.startViewTransition) {
    document.startViewTransition(() => {
      console.log('soft navigating to: ', n)
    })
  }
})
</script>

<template>
  <TooltipProvider>
    <div v-if="$route.data.frontmatter.layout === false">
      <Content :key="$route.path" />
    </div>
    <div v-else vaul-drawer-wrapper class="mx-auto w-full border-border/40 dark:border-border min-[1800px]:max-w-[1536px] min-[1800px]:border-x">
      <header class="sticky top-0 z-50 w-full border-b border-border/40 bg-background/95 backdrop-blur supports-[backdrop-filter]:bg-background/60 dark:border-border">
        <div class="flex h-14 items-center px-4">
          <div class="mr-4 md:mr-1 hidden md:flex">
            <Logo />

            <nav class="flex items-center gap-4 text-sm xl:gap-6">
              <a
                v-for="route in docsConfig.mainNav"
                :key="route.title"
                :href="route.href"
                :target="route.external ? '_target' : undefined"
                :class="cn('transition-colors hover:text-foreground/80', $route.path === `${route.href}.html` ? 'text-foreground' : 'text-foreground/80')"
              >
                {{ route.title }}
              </a>
            </nav>
          </div>
          <MobileNav />

          <div class="flex flex-1 items-center justify-between space-x-2 md:justify-end">
            <div class="w-full flex-1 md:w-auto md:flex-none">
              <Button
                variant="outline"
                class="relative h-8 w-full justify-start rounded-[0.5rem] bg-muted/50 text-sm font-normal text-muted-foreground shadow-none sm:pr-12 md:w-40 lg:w-56 xl:w-64"
                @click="isOpen = true"
              >
                <span class="hidden lg:inline-flex">Search documentation...</span>
                <span class="inline-flex lg:hidden">Search...</span>
                <Kbd :size="'xs'" class="pointer-events-none absolute right-[0.3rem] top-[0.3rem] hidden h-5 select-none items-center gap-1 rounded border bg-muted px-1.5 font-mono text-[10px] font-medium opacity-100 sm:flex">
                  ⌘ K
                </Kbd>
              </Button>
            </div>

            <nav class="flex items-center gap-0.5">
              <ThemePopover />

              <CodeConfigCustomizer />

              <Button
                v-for="link in links"
                :key="link.name"
                as="a"
                class="w-8 h-8"
                :href="link.href" target="_blank"
                :variant="'ghost'"
                :size="'icon'"
              >
                <component :is="link.icon" class="w-4 h-4" />
              </Button>

              <Button
                class="w-8 h-8"
                aria-label="Toggle dark mode"
                :variant="'ghost'"
                :size="'icon'"
                @click="toggleDark()"
              >
                <component
                  :is="isDark ? SunIcon : MoonIcon"
                  class="w-4 h-4 text-foreground"
                />
              </Button>
            </nav>
          </div>
        </div>
      </header>

      <div class="flex-1  bg-background">
        <Transition name="fade" mode="out-in">
          <component :is="'docs'" v-if="$route.path.includes('docs')">
            <Transition name="fade" mode="out-in">
              <Content :key="$route.path" />
            </Transition>
          </component>
          <component :is="'examples'" v-else-if="$route.path.includes('examples')">
            <Transition name="fade" mode="out-in">
              <Content :key="$route.path" />
            </Transition>
          </component>
          <component :is="frontmatter.layout" v-else-if="frontmatter.layout">
            <slot />
          </component>
          <main v-else class="flex-1">
            <Transition name="fade" mode="out-in">
              <Content :key="$route.path" />
            </Transition>
          </main>
        </Transition>
      </div>

      <footer class="border-t border-border/40 py-6 dark:border-border md:px-8 md:py-0">
        <div class="container flex flex-col items-center justify-between gap-4 md:h-24 md:flex-row">
          <p class="text-balance text-center text-sm leading-loose text-muted-foreground md:text-left">
            <span>
              Built by
              <a
                href="https://twitter.com/shadcn"
                target="_blank"
                class="font-medium underline underline-offset-4"
              >
                shadcn</a>.
            </span>
            <span class="inline-block ml-1">
              Ported to Vue by
              <a
                href="https://github.com/unovue"
                target="_blank"
                class="font-medium underline underline-offset-4"
              >
                unovue
              </a>
            </span>.
            <span class="inline-block ml-1">
              The code source is available on
              <a
                href="https://github.com/unovue/shadcn-vue"
                target="_blank"
                class="font-medium underline underline-offset-4"
              >
                GitHub</a>.
            </span>
          </p>
        </div>
      </footer>

      <Dialog v-model:open="isOpen">
        <DialogContent class="p-0">
          <Command>
            <CommandInput placeholder="Type a command or search..." />
            <CommandEmpty>
              No results found.
            </CommandEmpty>
            <CommandList
              @escape-key-down=" isOpen = false"
            >
              <CommandGroup heading="Links">
                <CommandItem
                  v-for="item in docsConfig.mainNav"
                  :key="item.title"
                  :heading="item.title"
                  :value="item.title"
                  class="py-3"
                  @select="handleSelectLink(item)"
                >
                  <File class="mr-2 h-5 w-5" />
                  <span>{{ item.title }}</span>
                </CommandItem>
              </CommandGroup>
              <CommandSeparator />
              <CommandGroup v-for="item in docsConfig.sidebarNav" :key="item.title" :heading="item.title">
                <CommandItem
                  v-for="subItem in item.items"
                  :key="subItem.title"
                  :heading="subItem.title"
                  :value="subItem.title"
                  class="py-3"
                  @select="
                    handleSelectLink(subItem)"
                >
                  <Circle class="mr-2 h-4 w-4" />
                  <span>{{ subItem.title }}</span>
                </CommandItem>
              </CommandGroup>
              <CommandSeparator />
              <CommandGroup heading="Theme">
                <CommandItem
                  value="light-theme"
                  class="py-3"
                  @select="
                    () => {
                      isDark = false;
                      isOpen = false;
                    }
                  "
                >
                  <SunIcon class="mr-2 h-5 w-5" />
                  <span>Light Theme</span>
                </CommandItem>
                <CommandItem
                  value="dark-theme"
                  class="py-3"
                  @select="
                    () => {
                      isDark = true;
                      isOpen = false;
                    }
                  "
                >
                  <MoonIcon class="mr-2 h-5 w-5" />
                  <span>Dark Theme</span>
                </CommandItem>
              </CommandGroup>
            </CommandList>
          </Command>
        </DialogContent>
      </Dialog>
      <DefaultToaster />
      <NewYorkSonner class="pointer-events-auto" :theme="'system'" />
      <NewYorkToaster />
    </div>
  </TooltipProvider>
</template>

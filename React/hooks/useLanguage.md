```ts
import { useTranslation } from "react-i18next"
import { useCallback } from "react"
import { useLanguageStore } from "~/store/use-language-store"

export interface Language {
  code: string
  name: string
  nativeName: string
}

export const AVAILABLE_LANGUAGES: Language[] = [
  {
    code: "zh-Hans",
    name: "Chinese",
    nativeName: "中文",
  },
  {
    code: "en",
    name: "English",
    nativeName: "English", 
  }
]

export function useLanguage() {
  const { i18n } = useTranslation()
  const { setLanguage } = useLanguageStore()

  const currentLanguage = AVAILABLE_LANGUAGES.find(
    lang => lang.code === i18n.language
  ) || AVAILABLE_LANGUAGES[0]

  const changeLanguage = useCallback((languageCode: string) => {
    setLanguage(languageCode)
    i18n.changeLanguage(languageCode)
  }, [i18n, setLanguage])

  const isCurrentLanguage = useCallback((languageCode: string) => {
    return i18n.language === languageCode
  }, [i18n.language])

  return {
    currentLanguage,
    availableLanguages: AVAILABLE_LANGUAGES,
    changeLanguage,
    isCurrentLanguage,
    language: i18n.language
  }
}
```


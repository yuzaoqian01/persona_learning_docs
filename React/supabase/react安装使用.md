## 创建一个项目

这个可以将数据库拉到本地进行本地化项目开发

```js
supabase link --project-ref <project-id>
# You can get <project-id> from your project's dashboard URL: https://supabase.com/dashboard/project/<project-id>
supabase db pull
```

### 初始化 React

```js
// 我选择的react-router7
npm create vite@latest supabase-react -- --template react
cd supabase-react
```

项目创建后运行会报错 混合水合报错啥的 在html 标签上加入 suppressHydrationWarning 就不报错了 

配置环境文件

```js
// .env

VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=

```



创建supabase客户端

```js
// utils/supabase.ts (supabaseClient.ts)
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL
const supabasePublishableKey = import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY

export const supabase = createClient(supabaseUrl, supabasePublishableKey)
```

// 教程上的auth.tsx

```js
// Auth.jsx 
import { useState } from 'react'
import { supabase } from './supabaseClient'

export default function Auth() {
  const [loading, setLoading] = useState(false)
  const [email, setEmail] = useState('')

  const handleLogin = async (event) => {
    event.preventDefault()

    setLoading(true)
    // 这里直接调用邮箱登陆
    const { error } = await supabase.auth.signInWithOtp({ email })

    if (error) {
      alert(error.error_description || error.message)
    } else {
      alert('Check your email for the login link!')
    }
    setLoading(false)
  }

  return (
    <div className="row flex flex-center">
      <div className="col-6 form-widget">
        <h1 className="header">Supabase + React</h1>
        <p className="description">Sign in via magic link with your email below</p>
        <form className="form-widget" onSubmit={handleLogin}>
          <div>
            <input
              className="inputField"
              type="email"
              placeholder="Your email"
              value={email}
              required={true}
              onChange={(e) => setEmail(e.target.value)}
            />
          </div>
          <div>
            <button className={'button block'} disabled={loading}>
              {loading ? <span>Loading</span> : <span>Send magic link</span>}
            </button>
          </div>
        </form>
      </div>
    </div>
  )
}
```

const { error } = await supabase.auth.signInWithOtp({ email }) 执行后supabase 配置会发送给邮箱一个链接 链接是配置的邮箱回调地址带有code啥的 然后跳到你指定的地址后验证并登陆

```js
// 下载操作
const { data, error } = await supabase.storage.from('avatars').download(path)
      if (error) {
        throw error
      }
const url = URL.createObjectURL(data)
      
// 上传操作    
const { error: uploadError } = await supabase.storage.from('avatars').upload(filePath, file)
      if (uploadError) {
        throw uploadError
      }      
```


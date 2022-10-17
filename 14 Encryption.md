```ts
// helper/encryption.ts

import { createCipheriv, createDecipheriv, randomBytes, scrypt } from 'crypto'
import { promisify } from 'util'

const iv = randomBytes(16)
const password = process.env.ENCRYPTION_KEY || ''

export const encrypt = async (text: string) => {
  const key = (await promisify(scrypt)(password, 'salt', 32)) as Buffer
  const cipher = createCipheriv('aes-256-ctr', key, iv)
  const encryptedText = cipher.update(text, 'utf8', 'hex') + cipher.final('hex')
  return encryptedText
}

export const decrypt = async (text: string) => {
  const key = (await promisify(scrypt)(password, 'salt', 32)) as Buffer
  const decipher = createDecipheriv('aes-256-ctr', key, iv)
  const decryptedText = decipher.update(text, 'hex', 'utf8') + decipher.final('utf8')
  return decryptedText
}
```

```
npm i stripe
```

```ts
import Stripe from 'stripe'

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY as string, {
  apiVersion: '2022-08-01',
})

const storeItems = [
  { id: 1, priceInCents: 50000, name: 'Item 1' },
  { id: 2, priceInCents: 100000, name: 'Item 2' },
]

interface storeItemInterface {
  id: number
  quantity: number
}

app.post('/checkout', async (req: Request, res: Response) => {
  const session = await stripe.checkout.sessions.create({
    payment_method_types: ['card'],
    mode: 'payment',
    line_items: req.body.items.map((item: storeItemInterface) => {
      const storeItem = storeItems.find(storeItem => storeItem.id === item.id)
      if (!storeItem) return
      return {
        price_data: {
          currency: 'usd',
          product_data: {
            name: storeItem.name,
          },
          unit_amount: storeItem.priceInCents,
        },
        quantity: item.quantity,
      }
    }),
    success_url: `${process.env.SERVER_URL}success.html`,
    cancel_url: `${process.env.SERVER_URL}cancel.html`,
  })
  if (session.url) return res.redirect(session.url)
  res.status(500).json({ message: 'Something went wrong' })
})
```

This is a [Next.js](https://nextjs.org/) project bootstrapped with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app).

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `app/page.tsx`. The page auto-updates as you edit the file.

This project uses [`next/font`](https://nextjs.org/docs/basic-features/font-optimization) to automatically optimize and load Inter, a custom Google Font.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js/) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/deployment) for more details.


--daisyui -  library with tailwindcss
--install daisyuui by using npm i -D daisyui@latest
*** go through daisyui.com websitr for information ***


01.Introduction
02.Install Tools
03.Create Next App
04.Publish to Github
05.List Products
    1.create product type
    2.add data.ts
    3.add images
    4.render products
06.Create Product Details
    1.create product page
    2.create 3 columns
    3.show image in first column
    4.show product info in second column
    5.show add to cart action on third column   
    6.add styles

07.***Handle Add To Cart***

    1. lib/utils.ts

    ```ts
    export const round2 = (num: number) =>
        Math.round((num + Number.EPSILON) * 100) / 100
    ```

    2. lib/models/OrderModel.ts

    ```ts
    export type OrderItem = {
        name: string
        slug: string
        qty: number
        image: string
        price: number
        color: string
        size: string
    }
    ```

    3. npm install zustand --it's a state manager based on hooks

    4. lib/hooks/useCartStore.ts

    ```tsx
                import { create } from 'zustand'
                import { persist } from 'zustand/middleware'
                import { round2 } from '../utils'
                import { OrderItem} from '../models/OrderModel'

                const initialState: Cart = {
                items: [],
                itemsPrice: 0,
                taxPrice: 0,
                shippingPrice: 0,
                totalPrice: 0,

                }

                export const cartStore = create<Cart>()(
                persist(() => initialState, {
                name: 'cartStore',
                })
                )

                export default function useCartService() {
                const {
                items, itemsPrice, taxPrice,  shippingPrice, totalPrice,
                } = cartStore()

                return {
                items,
                itemsPrice,
                taxPrice,
                shippingPrice,
                totalPrice,
                increase: (item: OrderItem) => {
                const exist = items.find(
                (x) =>
                x.slug === item.slug
                )
                const updatedCartItems = exist
                ? items.map((x) =>
                x.slug === item.slug
                ? { ...exist,  qty: exist.qty + 1} : x )
                : [...items, { ...item, qty: 1 }]

                    const { itemsPrice, shippingPrice, taxPrice, totalPrice } =
                        calcPrice(updatedCartItems)
                    cartStore.setState({
                        items: updatedCartItems,
                        itemsPrice,
                        shippingPrice,
                        taxPrice,
                        totalPrice,
                    })
                    },
                    init: () => cartStore.setState(initialState),
                }
                }

                const calcPrice = (items: OrderItem[]) => {
                const itemsPrice = round2(
                items.reduce((acc, item) => acc + item.price _ item.qty, 0)
                ),
                shippingPrice = round2(itemsPrice > 100 ? 0 : 100),
                taxPrice = round2(Number(0.15 _ itemsPrice)),
                totalPrice = round2(itemsPrice + shippingPrice + taxPrice)
                return { itemsPrice, shippingPrice, taxPrice, totalPrice }
                }

                type Cart = {
                items: OrderItem[]
                itemsPrice: number
                taxPrice: number
                shippingPrice: number
                totalPrice: number
                }
    ```

    5. components/product/AddToCart.tsx

    ```tsx
    'use client'
    import useCartService from '@/lib/hooks/useCartStore'
    import { OrderItem } from '@/lib/models/OrderModel'
    import { useRouter } from 'next/navigation'
    import { useEffect, useState } from 'react'

    export default function AddToCart({ item }: { item: OrderItem }) {
        const router = useRouter()
        const { items, increase } = useCartService()
        const [existItem, setExistItem] = useState<OrderItem | undefined>()

        useEffect(() => {
        setExistItem(items.find((x) => x.slug === item.slug))
        }, [items])

        const addToCartHandler = () => {
        increase(item)
        }

        return existItem ? (
        <div>
            <button
            className="btn"
            type="button"
            onClick={() => decrease(existItem)}
            >
            -
            </button>
            <span className="px-2">{existItem.qty}</span>
            <button
            className="btn"
            type="button"
            onClick={() => increase(existItem)}
            >
            +
            </button>
        </div>
        ) : (
        <button
            className={'btn btn-primary w-full'}
            type="button"
            onClick={addToCartHandler}
        >
            Add to cart
        </button>
        )
    }
    ```

    6. app/(front)/product/[slug]/page.tsx

    ```tsx
    {
        product.countInStock !== 0 && (
        <div className="card-actions justify-center">
            <AddToCart item={product} />
        </div>
        )
    }
    ```

    7. components/header/Menu.tsx

    ```tsx
    'use client'
    import useCartService from '@/lib/hooks/useCartStore'
    import Link from 'next/link'
    import React, { useEffect, useState } from 'react'

    const Menu = () => {
        const { items } = useCartService()
        const [mounted, setMounted] = useState(false)
        useEffect(() => {
        setMounted(true)
        }, [])

        return (
        <>
            <div>
            <ul className="flex items-stretch">
                <li>
                <Link className="btn btn-ghost rounded-btn" href="/cart">
                    Cart
                    {mounted && items.length != 0 && (
                    <div className="badge badge-secondary">
                        {items.reduce((a, c) => a + c.qty, 0)}{' '}
                    </div>
                    )}
                </Link>
                </li>
                <li>
                <button className="btn btn-ghost rounded-btn" type="button">
                    Sign in
                </button>
                </li>
            </ul>
            </div>
        </>
        )
    }

    export default Menu
    ```

08.***Handle Remove Item From Cart***

    1. lib/useCartService.ts

    ```ts
    decrease: (item: OrderItem) => {
        const exist = items.find(
            (x) =>
            x.slug === item.slug
        )
        if (!exist) return
        const updatedCartItems =
            exist.qty === 1
            ? items.filter((x: OrderItem) => x.slug !== item.slug)
            : items.map((x) =>
                item.slug
                    ? { ...exist, qty: exist.qty - 1 }
                    : x
                )

        const { itemsPrice, shippingPrice, taxPrice, totalPrice } =
            calcPrice(updatedCartItems)
        cartStore.setState({
            items: updatedCartItems,
            itemsPrice,
            shippingPrice,
            taxPrice,
            totalPrice,
        })
        },
    ```

    2. components/product/AddToCart.tsx

    ```tsx

        const { items, increase, decrease } = useCartService()

            <button
            className="btn"
            type="button"
            onClick={() => decrease(existItem)}
            >
            -
            </button>
    ```

# 09. Create Cart Page

    1. lib/useCartService.ts

    ```ts
    import { persist } from 'zustand/middleware'
    export const cartStore = create<Cart>()(
        persist(() => initialState, {
        name: 'cartStore',
        })
    )
    ```

    2. app/(front)/cart/CartDetails.tsx

    ```tsx
    'use client'
    import Image from 'next/image'
    import Link from 'next/link'
    import { useRouter } from 'next/navigation'
    import useCartService from '@/lib/hooks/useCartStore'
    import { useEffect, useState } from 'react'

    export default function CartDetails() {
        const router = useRouter()
        const { items, itemsPrice, decrease, increase } = useCartService()

        const [mounted, setMounted] = useState(false)
        useEffect(() => {
        setMounted(true)
        }, [])

        if (!mounted) return <></>

        return (
        <>
            <h1 className="py-4 text-2xl">Shopping Cart</h1>

            {items.length === 0 ? (
            <div>
                Cart is empty. <Link href="/">Go shopping</Link>
            </div>
            ) : (
            <div className="grid md:grid-cols-4 md:gap-5">
                <div className="overflow-x-auto md:col-span-3">
                <table className="table">
                    <thead>
                    <tr>
                        <th>Item</th>
                        <th>Quantity</th>
                        <th>Price</th>
                    </tr>
                    </thead>
                    <tbody>
                    {items.map((item) => (
                        <tr key={item.slug}>
                        <td>
                            <Link
                            href={`/product/${item.slug}`}
                            className="flex items-center"
                            >
                            <Image
                                src={item.image}
                                alt={item.name}
                                width={50}
                                height={50}
                            ></Image>
                            <span className="px-2">
                                {item.name} ({item.color} {item.size})
                            </span>
                            </Link>
                        </td>
                        <td>
                            <button
                            className="btn"
                            type="button"
                            onClick={() => decrease(item)}
                            >
                            -
                            </button>
                            <span className="px-2">{item.qty}</span>
                            <button
                            className="btn"
                            type="button"
                            onClick={() => increase(item)}
                            >
                            +
                            </button>
                        </td>
                        <td>${item.price}</td>
                        </tr>
                    ))}
                    </tbody>
                </table>
                </div>
                <div>
                <div className="card bg-base-300">
                    <div className="card-body">
                    <ul>
                        <li>
                        <div className="pb-3 text-xl">
                            Subtotal ({items.reduce((a, c) => a + c.qty, 0)}) : $
                            {itemsPrice}
                        </div>
                        </li>
                        <li>
                        <button
                            onClick={() => router.push('/shipping')}
                            className="btn btn-primary w-full"
                        >
                            Proceed to Checkout
                        </button>
                        </li>
                    </ul>
                    </div>
                </div>
                </div>
            </div>
            )}
        </>
        )
    }
    ```

    3. app/(front)/cart/page.tsx

    ```tsx
    import CartDetails from './CartDetails'

    export const metadata = {
        title: 'Shopping Cart',
    }
    export default function CartPage() {
        return <CartDetails />
    }
    ```
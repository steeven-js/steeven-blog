---
external: false
title: "React Firebase Lister"
description: "Comment lister des données de firebase dans un projet React?"
date: 2023-08-23
---

## React avec Vite + Tailwindcss

Installer `reactJs` avec `vite` . [vite / reactJs](https://vitejs.dev/guide/).

```html
npm create vite@latest
```
Installer `TailwindCss` après avoir créer le projet reactJs
[TailwindCss](https://tailwindcss.com/docs/guides/vite).

```html
npm install -D tailwindcss postcss autoprefixer  
npx tailwindcss init -p
```

## Configuration du projet
-Installer react Router dom
```html
npm i react-router-dom
```
-Installer DaisyUi (obtionnel)

## Initialisation du projet Main.jsx

- react-router-dom et entourer `<App />` avec `<BrowserRouter></BrowserRouter>`
```js
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import { BrowserRouter } from 'react-router-dom'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>,
);
```
- Créer un fichier `firebase.js` à la racine du projet

## Firebase
-Créer un projet Firebase  
-Créer un database firebase (Firestore Database)  
-Copier coller les information de la db dans firebase.js

```js
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { getFirestore } from "@firebase/firestore";
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
const firebaseConfig = {
    apiKey:  
    authDomain:  
    projectId:  
    storageBucket:  
    messagingSenderId:  
    appId:  
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);

// Fire Store
export const firestore = getFirestore(app)

export default app;
```

-Créer une collection et des documents de la collection

## App.jsx

-Créer un composant pour lister les produits (PopularProducts)  
-Créer un composant pour afficher les détails d'un produit (ShoesDetails)

```js
import { Routes, Route } from 'react-router-dom'
import './index.css'

import PopularProducts from "./sections/PopularProducts"
import ShoesDetails from './sections/ShoesDetails'

function App() {

  return (
    <main>
      <section className='padding'>
        <Routes>
          <Route path='/' element={<PopularProducts />} />
          <Route path='/details/:id' element={<ShoesDetails />} />
        </Routes>
      </section>
    </main >

  )
}

export default App
```
## PopularProducts.jsx

```js
import React, { useEffect, useState } from 'react'
import { query, collection, getDocs } from "firebase/firestore";
import { firestore } from '../firebase';
import Shoes from '../common/Shoes';

const PopularProducts = () => {

    // État local pour stocker la liste des chaussures
    const [shoes, setShoes] = useState([])

    // Fonction pour récupérer la liste des chaussures depuis Firestore
    const getShoes = async () => {
        // console.log('start');

        // Création de la requête pour obtenir la collection "Nike"
        const rqShoes = query(collection(firestore, "Nike"))
        // console.log(rqShoes);

        // Récupération des documents de la collection "Nike"
        const snapShoes = await getDocs(rqShoes);
        // console.log(snapShoes);

        if (!snapShoes.empty) {
            // Formatage de la liste des chaussures et stockage dans dataShoes
            const dataShoes = snapShoes.docs.map(item => ({
                id: item.id,
                ...item.data()
            }))

            // console.log("dataShoes :", dataShoes);

            // Mise à jour de la liste des chaussures avec setChaussures
            setShoes(dataShoes);
        }

    }

    useEffect(() => {
        getShoes();
    }, []);

    return (
        <section id='products' className='max-container max-sm:mt-12'>
            <div className='flex flex-col justify-start gap-5'>
                <h2 className='text-4xl font-palanquin font-bold'>
                    Nos <span className='text-coral-red'> Chaussures </span> Populaires
                </h2>
                <p className='lg:max-w-lg mt-2 font-montserrat text-slate-gray'>
                    Chaussures et baskets pour homme.
                </p>
            </div>

            <div className='mt-16 grid lg:grid-cols-4 md:grid-cols-3 sm:grid-cols-2 grid-cols-1 sm:gap-6 gap-14'>

                {shoes.map(itemShoes => (

                    <Shoes key={itemShoes.id} data={itemShoes} />

                ))}

            </div>
        </section>
    )
}

export default PopularProducts
```
-Créer un composant qui sera boucler avec les informations de la base de donnée.

## Shoes.jsx
```js
import React from 'react';
import { Link } from 'react-router-dom';

const Shoes = ({ data }) => {
    return (
        <div>
            <Link to={`/details/${data.id}`}>
                <img src={data.image} alt={data.name} className='w-[282px] h-[282px]' />
                <p className='lg:max-w-lg mt-2 font-montserrat text-slate-gray'> {data.name} | 
                    <span className='text-coral-red' >  {data.price / 100}€</span> 
                </p>
            </Link>
        </div>
    );
};

export default Shoes;
```
-Créer un composant pour afficher les détails d'un produit.

## ShoesDetails.jsx
```js
import { collection, doc, getDoc } from 'firebase/firestore';
import React, { useEffect, useState } from 'react'
import { firestore } from '../firebase'
import { useParams } from 'react-router-dom';

const ShoesDetails = () => {
    const { id } = useParams(); // Obtenez l'ID à partir des paramètres de l'URL
    const [shoe, setShoe] = useState(null); // État local pour stocker les informations de la chaussure

    const getShoe = async (shoeId) => {
        const shoeRef = doc(collection(firestore, 'Nike'), shoeId); // 'Nike' est le nom de la collection dans Firestore

        const snapShoe = await getDoc(shoeRef);

        if (snapShoe.exists()) {
            // Récupérez les informations de la chaussure
            const shoeData = snapShoe.data();
            setShoe(shoeData);
        }
    }

    useEffect(() => {
        getShoe(id); // Passez l'ID à la fonction getShoe
    }, [id]);

    if (!shoe) {
        return <div>Loading...</div>; // Ajoutez une indication de chargement
    }

    return (

        <section
            id='about-us'
            className='flex justify-between items-center max-lg:flex-col gap-10 w-full max-container'
        >
            <div className='flex-1 flex justify-center items-center'>
                <img
                    src={shoe.image}
                    alt={shoe.name}
                    className='object-contain w-[500px] h-[500px]'
                />
            </div>

            <div className='flex flex-1 flex-col'>
                <h2 className='font-palanquin capitalize text-4xl lg:max-w-lg font-bold'>
                    {shoe.name}
                </h2>
                <p className='mt-4 lg:max-w-lg info-text'>
                    {shoe.description}
                </p>
                <p className='mt-6 lg:max-w-lg info-text'>
                    {shoe.price / 100}€
                </p>
                <div className='mt-11'>

                    <a href="#" class="bg-orange-500 hover:bg-orange-600 text-white font-semibold py-2 px-4 rounded">
                        Ajouter au panier
                    </a>

                </div>
            </div>

        </section>
    );
}

export default ShoesDetails;

```

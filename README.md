# Penjelasan Autentikasi Login dengan Google dan CRUD 

- Nama : Fawwaz Afkar Muzakky
- NIM : H1D022067
- Shift(KRS/Baru) : B/C


## Langkah-langkah Autentikasi Login

`(Untuk penjelasan CRUD bisa scroll sampai Bawah)`

### 1. Konfigurasi Firebase
Pertama, konfigurasi Firebase, Buat file `firebase.ts` di folder `src/utils` dan tambahkan konfigurasi Firebase:

```typescript
// src/utils/firebase.ts
import { initializeApp } from "firebase/app";
import { getAuth, GoogleAuthProvider } from 'firebase/auth';

const firebaseConfig = {
    apiKey: "API_KEY",
    authDomain: "AUTH_DOMAIN",
    projectId: "PROJECT_ID",
    storageBucket: "STORAGE_BUCKET",
    messagingSenderId: "MESSAGING_SENDER_ID",
    appId: "APP_ID"
};

const firebase = initializeApp(firebaseConfig);
const auth = getAuth(firebase);
const googleProvider = new GoogleAuthProvider();

export { auth, googleProvider };
```

### 2. Setup Pinia Store untuk Autentikasi
Buat store Pinia untuk mengelola status autentikasi pengguna di `src/stores/auth.ts`:

```typescript
// src/stores/auth.ts
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import router from '@/router';
import { auth } from '@/utils/firebase';
import { GoogleAuthProvider, onAuthStateChanged, signInWithCredential, signOut, User } from 'firebase/auth';
import { GoogleAuth } from '@codetrix-studio/capacitor-google-auth';
import { alertController } from '@ionic/vue';

export const useAuthStore = defineStore('auth', () => {
    const user = ref<User | null>(null);
    const isAuth = computed(() => user.value !== null);

    const loginWithGoogle = async () => {
        try {
            await GoogleAuth.initialize({
                clientId: '556382459801-tdkrvbflmkf7jo30rh9qpa7eqb9iq4je.apps.googleusercontent.com',
                scopes: ['profile', 'email'],
                grantOfflineAccess: true,
            });

            const googleUser = await GoogleAuth.signIn();
            const idToken = googleUser.authentication.idToken;
            const credential = GoogleAuthProvider.credential(idToken);
            const result = await signInWithCredential(auth, credential);
            user.value = result.user;
            router.push("/home");
        } catch (error) {
            console.error("Google sign-in error:", error);
            const alert = await alertController.create({
                header: 'Login Gagal!',
                message: 'Terjadi kesalahan saat login dengan Google. Coba lagi.',
                buttons: ['OK'],
            });
            await alert.present();
            throw error;
        }
    };

    const logout = async () => {
        try {
            await signOut(auth);
            await GoogleAuth.signOut();
            user.value = null;
            router.replace("/login");
        } catch (error) {
            console.error("Sign-out error:", error);
            throw error;
        }
    };

    onAuthStateChanged(auth, (currentUser) => {
        user.value = currentUser;
    });

    return { user, isAuth, loginWithGoogle, logout };
});
```

### 3. Implementasi Halaman Login
Buat halaman login di `src/pages/LoginPage.vue`:

```vue
<template>
    <ion-page>
        <ion-content :fullscreen="true">
            <div id="container">
                <ion-text style="margin-bottom: 20px; text-align: center;">
                    <h1>Praktikum Pemrograman Mobile</h1>
                </ion-text>
                <ion-button @click="login" color="light">
                    <ion-icon slot="start" :icon="logoGoogle"></ion-icon>
                    <ion-label>Sign In with Google</ion-label>
                </ion-button>
            </div>
        </ion-content>
    </ion-page>
</template>

<script setup lang="ts">
import { IonContent, IonPage, IonButton, IonIcon, IonText, IonLabel } from '@ionic/vue';
import { logoGoogle } from 'ionicons/icons';
import { useAuthStore } from '@/stores/auth';

const authStore = useAuthStore();

const login = async () => {
    await authStore.loginWithGoogle();
};
</script>

<style>
#container {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 100%;
}

ion-button {
    --border-radius: 8px;
}
</style>
```

### 4. Implementasi Halaman Profil
Buat halaman profil di `src/pages/ProfilePage.vue`:

```vue
<template>
    <ion-page>
        <ion-header :translucent="true">
            <ion-toolbar>
                <ion-title>Profile</ion-title>
                <ion-button slot="end" fill="clear" @click="logout" style="--color: gray;">
                    <ion-icon slot="end" :icon="exit"></ion-icon>
                    <ion-label>Logout</ion-label>
                </ion-button>
            </ion-toolbar>
        </ion-header>
        <ion-content :fullscreen="true">
            <div id="avatar-container">
                <ion-avatar>
                    <img alt="Avatar" :src="userPhoto" @error="handleImageError" />
                </ion-avatar>
            </div>
            <ion-list>
                <ion-item>
                    <ion-input label="Nama" :value="user?.displayName" :readonly="true"></ion-input>
                </ion-item>
                <ion-item>
                    <ion-input label="Email" :value="user?.email" :readonly="true"></ion-input>
                </ion-item>
            </ion-list>
            <TabsMenu />
        </ion-content>
    </ion-page>
</template>

<script setup lang="ts">
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonInput, IonItem, IonList, IonLabel, IonIcon, IonButton, IonAvatar } from '@ionic/vue';
import { exit } from 'ionicons.icons';
import { computed, ref } from 'vue';
import TabsMenu from '@/components/TabsMenu.vue';
import { useAuthStore } from '@/stores/auth';

const authStore = useAuthStore();
const user = computed(() => authStore.user);

const logout = () => {
    authStore.logout();
};

const userPhoto = ref(user.value?.photoURL || 'https://ionicframework.com/docs/img/demos/avatar.svg');

function handleImageError() {
    userPhoto.value = 'https://ionicframework.com/docs/img/demos/avatar.svg';
}
</script>

<style scoped>
#avatar-container {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 20px 0;
}

#avatar-icon {
    width: 80px;
    height: 80px;
}
</style>
```

### Penjelasan Proses Autentikasi Login

1. **Inisialisasi Firebase**:
    - Pertama, kita perlu mengatur Firebase di aplikasi kita. Caranya dengan membuat file bernama `firebase.ts` di dalam folder `src/utils`.
    - Di dalam file ini, kita akan mengatur konfigurasi Firebase menggunakan `initializeApp` dari Firebase SDK. Selain itu, kita juga akan menginisialisasi `auth` dan `googleProvider` yang akan digunakan untuk login dengan Google.

2. **Setup Pinia Store untuk Autentikasi**:
    - Selanjutnya, kita buat store Pinia di `src/stores/auth.ts` untuk mengelola status login pengguna.
    - Di dalam store ini, ada state `user` yang menyimpan informasi pengguna dan `isAuth` untuk mengecek apakah pengguna sudah login atau belum.
    - Ada fungsi `loginWithGoogle` yang digunakan untuk memulai proses login dengan Google. Fungsi ini akan menginisialisasi GoogleAuth, meminta pengguna untuk login, dan mendapatkan `idToken`.
    - `idToken` ini kemudian digunakan untuk membuat kredensial Google yang digunakan untuk login ke Firebase dengan `signInWithCredential`.
    - Jika login berhasil, informasi pengguna akan disimpan di state `user` dan pengguna akan diarahkan ke halaman `/home`.
    - Ada juga fungsi `logout` yang digunakan untuk keluar dari aplikasi, menghapus status login pengguna, dan mengarahkan pengguna kembali ke halaman login.

3. **Implementasi Halaman Login**:
    - Kita buat halaman login di `src/pages/LoginPage.vue`.
    - Di halaman ini, ada tombol yang akan memanggil fungsi `login` dari store Pinia ketika diklik.
    - Fungsi `login` ini akan memanggil `loginWithGoogle` dari store untuk memulai proses login.

4. **Implementasi Halaman Profil**:
    - Kita buat halaman profil di `src/views/ProfilePage.vue`.
    - Di halaman ini, kita akan menampilkan informasi pengguna seperti nama dan email yang diambil dari state `user` di store Pinia.
    - Halaman ini juga memiliki tombol logout yang akan memanggil fungsi `logout` dari store untuk keluar dari aplikasi.


### Screenshot

Berikut adalah screenshot dari aplikasi yang telah selesai:

![Screenshot](./screenshots/login.png)
![ss home](./screenshots/home.png)
![ss profile](./screenshots/profile.png)

## Penjelasan Proses CRUD ke Firebase

Aplikasi ini menggunakan Firebase Firestore sebagai backend untuk menyimpan data Todo. 

1. Struktur Data Todo

Data Todo memiliki struktur sebagai berikut:
```typescript
export interface Todo {
    id?: string;
    title: string;
    description: string;
    status: boolean;
    createdAt: Timestamp;
    updatedAt: Timestamp;
}
```

2. Mengambil Referensi Koleksi Todo

Setiap operasi CRUD dimulai dengan mendapatkan referensi ke koleksi Todo di Firestore:
```typescript
    getTodoRef() {
    const uid = auth.currentUser?.uid;
    if (!uid) throw new Error('User not authenticated');
    return collection(db, 'users', uid, 'todos');
}
```

Referensi ini digunakan untuk mengakses dokumen Todo milik pengguna yang sedang login.

3. Operasi Create

![add](./img/addtodo.png)

Untuk menambahkan Todo baru, aplikasi memanggil addTodo:
```typescript
async addTodo(todo: Omit<Todo, 'id'>) {
    const todoRef = this.getTodoRef();
    const docRef = await addDoc(todoRef, {
        ...todo,
        status: false,
        createdAt: Timestamp.now(),
        updatedAt: Timestamp.now()
    });
    return docRef.id;
}
```

Data Todo dikirim ke Firestore dengan menambahkan dokumen baru ke koleksi.

4. Operasi Read

![Screenshot](./img/hometodo.png)

Untuk membaca semua Todo, aplikasi memanggil getTodos:
```typescript
async getTodos(): Promise<Todo[]> {
    const todoRef = this.getTodoRef();
    const q = query(todoRef, orderBy('updatedAt', 'desc'));
    const snapshot = await getDocs(q);
    return snapshot.docs.map((doc) => ({
        id: doc.id,
        ...doc.data()
    } as Todo));
}
```

Data Todo diambil dari Firestore dan diurutkan berdasarkan updatedAt.

5. Operasi Update

![Screenshot](./img/edittodo.png)

Untuk memperbarui Todo, aplikasi memanggil updateTodo:
```typescript
async updateTodo(id: string, todo: Partial<Todo>) {
    const todoRef = this.getTodoRef();
    const docRef = doc(todoRef, id);
    await updateDoc(docRef, {
        ...todo,
        updatedAt: Timestamp.now()
    });
}
```

Dokumen Todo yang ada diperbarui dengan data baru.

6. Operasi Delete

before
![Screenshot](./img/deletetodo.png)

after
![Screenshot](./img/afterdelete.png)

Untuk menghapus Todo, aplikasi memanggil deleteTodo:
```typescript
async deleteTodo(id: string) {
    const todoRef = this.getTodoRef();
    const docRef = doc(todoRef, id);
    await deleteDoc(docRef);
}
```

Dokumen Todo dihapus dari koleksi.

7. Integrasi dengan Komponen Vue

Komponen InputModal.vue digunakan untuk menambah atau mengedit Todo. Data yang diinput oleh pengguna dikirim ke Firestore melalui event submit:
```typescript
const input = () => {
    emit('submit', localTodo.value);
    cancel();
}
```
Komponen `HomePage.vue` menampilkan daftar Todo dan menangani operasi CRUD:

- `loadTodos` memuat data Todo dari Firestore.
- `handleSubmit` menangani penambahan atau pengeditan Todo.
- `handleDelete` menangani penghapusan Todo.
- `handleStatus` menangani perubahan status Todo.

*** asset lengkap screenshot ada di folder img

### Penjelasan Logic CRUD

1. Create (Menambahkan Todo)
- InputModal.vue: Pengguna memasukkan data Todo melalui modal input.
- handleSubmit di HomePage.vue: Fungsi ini dipanggil saat pengguna menekan tombol submit pada modal. Fungsi ini memanggil addTodo dari firestoreService.
- firestoreService.addTodo di firestore.ts: Fungsi ini menambahkan Todo baru ke Firestore dengan menggunakan addDoc.

2. Read (Membaca Todo)
- loadTodos di HomePage.vue: Fungsi ini dipanggil saat halaman dimuat dan saat pengguna melakukan refresh Fungsi ini memanggil getTodos dari firestoreService.
- firestoreService.getTodos di firestore.ts: Fungsi ini mengambil semua Todo dari Firestore dengan menggunakan getDocs dan mengurutkannya berdasarkan updatedAt.

3. Update (Mengedit Todo)
- handleEdit di HomePage.vue: Fungsi ini dipanggil saat pengguna menekan tombol edit pada Todo. Fungsi ini membuka modal input dengan data Todo yang akan diedit.
- handleSubmit di HomePage.vue: Fungsi ini juga menangani pengeditan Todo. Jika editingId tidak null, fungsi ini memanggil updateTodo dari firestoreService.
- firestoreService.updateTodo di firestore.ts: Fungsi ini memperbarui Todo di Firestore dengan menggunakan updateDoc.

4. Delete (Menghapus Todo)
- handleDelete di HomePage.vue: Fungsi ini dipanggil saat pengguna menekan tombol delete pada Todo. Fungsi ini memanggil deleteTodo dari firestoreService.
- firestoreService.deleteTodo di firestore.ts: Fungsi ini menghapus Todo dari Firestore dengan menggunakan deleteDoc.

5. Update Status (Mengubah Status Todo)
- handleStatus di HomePage.vue: Fungsi ini dipanggil saat pengguna menekan tombol untuk mengubah status Todo. Fungsi ini memanggil updateStatus dari firestoreService.
- firestoreService.updateStatus di firestore.ts: Fungsi ini memperbarui status Todo di Firestore dengan menggunakan updateDoc.

### Flow CRUD
- Pengguna menambahkan/mengedit Todo: Data Todo dikirim ke Firestore melalui firestoreService.
- Pengguna membaca Todo: Data Todo diambil dari Firestore dan ditampilkan di aplikasi.
- Pengguna menghapus Todo: Todo dihapus dari Firestore melalui firestoreService.
- Pengguna mengubah status Todo: Status Todo diperbarui di Firestore melalui firestoreService.
import { useEffect, useState } from "react";
import { motion } from "framer-motion";
import {
  ArrowLeft,
  Send,
  Heart,
  User,
  BookOpen,
  MessageCircle,
  PenLine,
  Shield,
} from "lucide-react";

import { initializeApp, getApps } from "firebase/app";
import {
  getAuth,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  onAuthStateChanged,
  signOut,
  User as FirebaseUser,
} from "firebase/auth";
import {
  getFirestore,
  collection,
  addDoc,
  onSnapshot,
  query,
  orderBy,
  serverTimestamp,
} from "firebase/firestore";
import {
  getStorage,
  ref,
  uploadBytes,
  getDownloadURL,
} from "firebase/storage";

/* ================= FIREBASE ================= */
const firebaseConfig = {
  apiKey: "AIzaSyD1TRiz76fI2va46LlrMo6KMnHUjKHaxBM",
  authDomain: "fole-c2b42.firebaseapp.com",
  projectId: "fole-c2b42",
  storageBucket: "fole-c2b42.appspot.com",
  messagingSenderId: "867442032494",
  appId: "1:867442032494:web:45122f57ed416a65cd0b7f",
};

const app = getApps().length ? getApps()[0] : initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const storage = getStorage(app);

const SUPER_ADMIN = "keisikuqo22@gmail.com";

type Page =
  | "login"
  | "register"
  | "home"
  | "chat"
  | "creations"
  | "word"
  | "authors"
  | "profile"
  | "admin";

export default function App() {
  const [page, setPage] = useState<Page>("login");
  const [user, setUser] = useState<FirebaseUser | null>(null);

  /* AUTH */
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [nickname, setNickname] = useState("");
  const [role, setRole] = useState<"nxenes" | "mesues">("nxenes");
  const [grade, setGrade] = useState("6");
  const [error, setError] = useState("");

  /* CHAT */
  const [chat, setChat] = useState<any[]>([]);
  const [chatText, setChatText] = useState("");

  /* CREATIONS */
  const [posts, setPosts] = useState<any[]>([]);
  const [postText, setPostText] = useState("");
  const [postFile, setPostFile] = useState<File | null>(null);

  /* WORD */
  const [dictionary, setDictionary] = useState<any[]>([]);
  const [w, setW] = useState("");
  const [e, setE] = useState("");
  const [ex, setEx] = useState("");

  /* AUTHORS */
  const [authors, setAuthors] = useState<any[]>([]);
  const [aName, setAName] = useState("");
  const [aInfo, setAInfo] = useState("");
  const [aFile, setAFile] = useState<File | null>(null);

  /* ================= AUTH LISTENER ================= */
  useEffect(() => {
    return onAuthStateChanged(auth, (u) => {
      setUser(u);
      if (u) setPage("home");
    });
  }, []);

  /* ================= REALTIME ================= */
  useEffect(() => {
    if (!user) return;
    return onSnapshot(
      query(collection(db, "chat"), orderBy("time")),
      (s) => setChat(s.docs.map((d) => d.data()))
    );
  }, [user]);

  useEffect(() => {
    return onSnapshot(
      query(collection(db, "creations"), orderBy("created", "desc")),
      (s) => setPosts(s.docs.map((d) => d.data()))
    );
  }, []);

  useEffect(() => {
    return onSnapshot(collection(db, "dictionary"), (s) =>
      setDictionary(s.docs.map((d) => d.data()))
    );
  }, []);

  useEffect(() => {
    return onSnapshot(collection(db, "authors"), (s) =>
      setAuthors(s.docs.map((d) => d.data()))
    );
  }, []);

  /* ================= ACTIONS ================= */
  const login = async () => {
    try {
      await signInWithEmailAndPassword(auth, email, password);
      setPage("home");
    } catch {
      setError("Email ose password gabim");
    }
  };

  const register = async () => {
    try {
      await createUserWithEmailAndPassword(auth, email, password);
      await addDoc(collection(db, "profiles"), {
        email,
        nickname,
        role,
        grade,
      });
      setPage("home");
    } catch {
      setError("Email already in use");
    }
  };

  const sendMessage = async () => {
    if (!chatText) return;
    await addDoc(collection(db, "chat"), {
      text: chatText,
      author: user?.email,
      time: serverTimestamp(),
    });
    setChatText("");
  };

  const publishPost = async () => {
    let img = "";
    if (postFile) {
      const r = ref(storage, `posts/${Date.now()}`);
      await uploadBytes(r, postFile);
      img = await getDownloadURL(r);
    }
    await addDoc(collection(db, "creations"), {
      text: postText,
      image: img,
      author: user?.email,
      likes: 0,
      created: serverTimestamp(),
    });
    setPostText("");
    setPostFile(null);
  };

  const addWord = async () => {
    await addDoc(collection(db, "dictionary"), {
      word: w,
      explain: e,
      example: ex,
    });
    setW("");
    setE("");
    setEx("");
  };

  const addAuthor = async () => {
    let img = "";
    if (aFile) {
      const r = ref(storage, `authors/${Date.now()}`);
      await uploadBytes(r, aFile);
      img = await getDownloadURL(r);
    }
    await addDoc(collection(db, "authors"), {
      name: aName,
      info: aInfo,
      image: img,
    });
    setAName("");
    setAInfo("");
    setAFile(null);
  };

  /* ================= UI ================= */
  return (
    <div className="min-h-screen bg-red-50 p-4">
      <motion.h1 className="text-3xl text-center font-bold text-red-600 mb-4">
        FOL.E
      </motion.h1>

      {page !== "home" && page !== "login" && page !== "register" && (
        <button
          onClick={() => setPage("home")}
          className="flex items-center gap-2 text-red-600 mb-4"
        >
          <ArrowLeft /> Home
        </button>
      )}

      {/* LOGIN */}
      {page === "login" && (
        <div className="max-w-md mx-auto space-y-2">
          <input className="border p-2 w-full" placeholder="Email" onChange={(e) => setEmail(e.target.value)} />
          <input className="border p-2 w-full" type="password" placeholder="Password" onChange={(e) => setPassword(e.target.value)} />
          {error && <p className="text-red-600">{error}</p>}
          <button onClick={login} className="bg-red-500 text-white w-full py-2">Hyr</button>
          <button onClick={() => setPage("register")} className="w-full">Regjistrohu</button>
        </div>
      )}

      {/* REGISTER */}
      {page === "register" && (
        <div className="max-w-md mx-auto space-y-2">
          <input className="border p-2 w-full" placeholder="Nickname" onChange={(e) => setNickname(e.target.value)} />
          <select className="border p-2 w-full" onChange={(e) => setRole(e.target.value as any)}>
            <option value="nxenes">Nxënës</option>
            <option value="mesues">Mësues</option>
          </select>
          <select className="border p-2 w-full" onChange={(e) => setGrade(e.target.value)}>
            <option>6</option><option>7</option><option>8</option><option>9</option>
          </select>
          <input className="border p-2 w-full" placeholder="Email" onChange={(e) => setEmail(e.target.value)} />
          <input className="border p-2 w-full" type="password" placeholder="Password" onChange={(e) => setPassword(e.target.value)} />
          <button onClick={register} className="bg-red-500 text-white w-full py-2">Regjistrohu</button>
        </div>
      )}

      {/* HOME */}
      {page === "home" && (
        <div className="grid grid-cols-2 gap-3 max-w-xl mx-auto">
          <button onClick={() => setPage("chat")} className="card"><MessageCircle /> Chat</button>
          <button onClick={() => setPage("creations")} className="card"><PenLine /> Krijime</button>
          <button onClick={() => setPage("word")} className="card"><BookOpen /> Fjala</button>
          <button onClick={() => setPage("authors")} className="card"><User /> Autorë</button>
          <button onClick={() => setPage("profile")} className="card"><Shield /> Profili</button>
          {user?.email === SUPER_ADMIN && (
            <button onClick={() => setPage("admin")} className="card">Admin</button>
          )}
        </div>
      )}

      {/* CHAT */}
      {page === "chat" && (
        <div className="max-w-md mx-auto bg-white p-3 rounded">
          <div className="h-64 overflow-y-auto border mb-2 p-2">
            {chat.map((m, i) => (
              <p key={i}><b>{m.author}:</b> {m.text}</p>
            ))}
          </div>
          <div className="flex gap-2">
            <input className="border p-2 flex-1" value={chatText} onChange={(e) => setChatText(e.target.value)} />
            <button onClick={sendMessage}><Send /></button>
          </div>
        </div>
      )}

      {/* CREATIONS */}
      {page === "creations" && (
        <div className="max-w-md mx-auto space-y-2">
          <textarea className="border w-full p-2" value={postText} onChange={(e) => setPostText(e.target.value)} />
          <input type="file" onChange={(e) => setPostFile(e.target.files?.[0] || null)} />
          <button onClick={publishPost} className="bg-red-500 text-white px-3 py-1">Publiko</button>
          {posts.map((p, i) => (
            <div key={i} className="bg-white p-2 rounded">
              {p.image && <img src={p.image} />}
              <p>{p.text}</p>
              <p className="text-sm text-gray-500">{p.author}</p>
            </div>
          ))}
        </div>
      )}

      {/* WORD */}
      {page === "word" && (
        <div className="max-w-md mx-auto bg-white p-3 rounded">
          {dictionary.map((d, i) => (
            <div key={i}>
              <b>{d.word}</b>
              <p>{d.explain}</p>
              <i>{d.example}</i>
              <hr />
            </div>
          ))}
        </div>
      )}

      {/* AUTHORS */}
      {page === "authors" && (
        <div className="grid grid-cols-2 gap-2 max-w-md mx-auto">
          {authors.map((a, i) => (
            <div key={i} className="bg-white p-2 rounded text-center">
              <img src={a.image} className="h-24 mx-auto" />
              <b>{a.name}</b>
            </div>
          ))}
        </div>
      )}

      {/* ADMIN */}
      {page === "admin" && (
        <div className="max-w-md mx-auto bg-white p-3 space-y-2">
          <h2 className="font-bold">Panel Menaxheri</h2>
          <input className="border p-2 w-full" placeholder="Fjala" value={w} onChange={(e) => setW(e.target.value)} />
          <input className="border p-2 w-full" placeholder="Shpjegim" value={e} onChange={(e) => setE(e.target.value)} />
          <input className="border p-2 w-full" placeholder="Shembull" value={ex} onChange={(e) => setEx(e.target.value)} />
          <button onClick={addWord} className="bg-red-500 text-white w-full">Shto Fjalë</button>

          <input className="border p-2 w-full" placeholder="Autor" value={aName} onChange={(e) => setAName(e.target.value)} />
          <textarea className="border p-2 w-full" placeholder="Info" value={aInfo} onChange={(e) => setAInfo(e.target.value)} />
          <input type="file" onChange={(e) => setAFile(e.target.files?.[0] || null)} />
          <button onClick={addAuthor} className="bg-red-500 text-white w-full">Shto Autor</button>
        </div>
      )}

      {/* PROFILE */}
      {page === "profile" && (
        <div className="max-w-md mx-auto bg-white p-3">
          <p>{user?.email}</p>
          <button onClick={() => signOut(auth)} className="text-red-600">Dil</button>
        </div>
      )}
    </div>
  );
}

# Next Level Week 5
### 1 Create next app

create app:
```sh
npx create-next-app app-name
```
run app:
```sh
yarn dev
```

### 2 Install type script

```sh
yarn add typescript @types/react @types/node -D
```

### 3 Trabalhando com datas

```sh
yarn add date-fns
```
```javaScript
import format from 'date-fns/format';
import ptBR from 'date-fns/locale/pt-BR';

  const currentDate = format(new Date(), 'EEEEEE d MMMM', {
    locale: ptBR,
  })
```

### 4 Simular uma requisição para a API através do json-server

Caso exista um arquivo server.json onde tenham informações que simulam uma API é necessário instalar um pacote:

```sh
yarn add json-server -D
```
É necessário fazer uma alteração no package.json incluindo um novo script:

_package.json_
```json
"scripts": {
    "server": "json-server server.json -w -d 758 -p 3333"
  },
  ```
  Depois basta salvar o arquivo e rodar o servidor no terminal:

  ```sh
  yarn server
  ```

  ### Formas de fazer requisições para uma API: 
##### SPA - Single-Page-Application:
* Roda no JavaScript do browser;
_index.js_
```javaScript
import { useEffect } from 'react';

export default function Home(props) {
  useEffect(() => {
    fetch('http://localhost:3333/episodes')
    .then((response) => response.json())
    .then((data) => console.log(data))
  }, [])

  return (
    <div>
      <h1>Index</h1>
    </div>
  )
}
```

##### SSR - Server-Side-Rendering;
* Faz a requisição antes da página ser carregada;
_index.js_
```javaScript
export default function Home(props) {
  return (
    <div>
      <h1>Index</h1>
      <p>{JSON.stringify(props.episodes)}</p>
    </div>
  )
}

export async function getServerSideProps() {
  const response = await fetch('http://localhost:3333/episodes');
  const data = await response.json()
  return {
    props: {
      episodes: data,
    }
  }
}
```
#####  SSG - Static Site Generation;
* Faz a requisição para a API a cada 8 horas (revalidate: 60 * 60 * 8,);
* Funciona apenas em produção, para simular pode ser instalada uma build;
_index.js_
```javaScript
export default function Home(props) {
  return (
    <div>
      <h1>Index</h1>
      <p>{JSON.stringify(props.episodes)}</p>
    </div>
  )
}
export async function getStaticProps() {
  const response = await fetch('http://localhost:3333/episodes');
  const data = await response.json()
  return {
    props: {
      episodes: data,
    },
    revalidate: 60 * 60 * 8,
  }
}
```

### Instalação da build que simula o projeto em produção

* Se o projeto estiver rodando, a execução deve ser parada;
* O servidor da API deve estar rodando;

```sh
yarn build
```
Agora podemos ver em qual modelo o nosso projeto estã sendo renderizado;

##### Rodar projeto como em produção
  
```sh
yarn start
```

### 5 requisições com axios
É necessário fazer a instalação:
```sh
yarn add axios
```
* axios é uma biblioteca para fazer requisições HTTP assim como o fetch, mas o axios tem algumas funcionalidades a mais.
* entendimento por padrão do json;
* configurar uma url base, a url que se repete para todas as chamadas para a API;

##### Implementação da requisição com axios
Criar uma nova pasta dentro de src chamada 'services' e um arquivo api.ts ou js

Definição da url padrão:
_src/services/api.ts_
```javaScript
import axios from 'axios';

export const api = axios.create({
  baseURL: 'http://localhost:3333/',
})
```

Utilização da axios para a requisição
_src/pages/index.tsx_
```javaScript
import { GetStaticProps } from 'next';
import { api } from '../services/api';

type Episodes = {  
  id: string;
  title: string;
  numbers: string;
}

type HomeProps = {
  episodes: Episodes[];
}
export default function Home(props: HomeProps) {

  return (
    <div>
      <h1>Index</h1>
      <p>{JSON.stringify(props.episodes)}</p>
    </div>
  )
}

export const getStaticProps: GetStaticProps = async () => {
  const { data } = await api.get('episodes', {
    params: {
      _limit: 12,
      _sort: 'published_at',
      _order: 'desc'
    }
  });
  
  return {
    props: {
      episodes: data,
    },
    revalidate: 60 * 60 * 8,
  }
}
```

### 6 Formatação dos dados
A formatação de dados vindo de uma requisição de API devem ser formatados antes de serem renderizados,ou seja, antes de estarem no HTML para serem renderizados.
O ideal é ser feito logo após obter o retorno dos dados.

### 7 File System Rooting 
Para criar um arquivo que forma a rota da aplicação é necessário criar uma pasta dentro da pasta 'pages', nesse caso será 'episodes', dentro dessa pasta deve ser criado um arquivo o qual o nome do arquivo fica dentro de [], ex: [slug].tsx (pode ser qualquer nome).
Obs: Quando houver  [] a rotá é gerada dinâmicamente. caso seja uma rota estática não é necessário os [].

Agora o caminho da url para essa pasta fica nesse caso (http://localhost:3000/episodes/qualquercoisa), então qual quer coisa que for depois de '/episode' vai direcionar para a página 'episode'. Para isso precisamos pegar o slug que irá depois de '/episode';
A implementação completa está em '_src/pages/episodes/[slug].tsx_'

### Entendendo o getStaticPaths

Quando a rota possui [] é necessário informar o método getStaticPaths:

* Quando o paths: [] - nenhuma página é gerada estáticamente no momento da build
* O que determina o comportamento de quando uma pessoa acessa a página de um episódio que não foi gerado estaticamente é o 'fallback'.
* Quando fallback: 'blocking' - a pessoa só vai ser direcionada para a tela quando os dados já estiverem carregado, (gera novas páginas conforme as pessoas vão acessando e faz a revalidação dos dados)

_src/pages/episodes/[slug].tsx_
```javaScript
export const getStaticPaths: GetStaticPaths = async () => {
  return {
    paths: [],
    fallback: 'blocking'
  }
}
```

* Quando o paths: path -  Nesse caso o paths está recebendo dois ids que são responsáveis pelo caminho da página, o qual quando ocorrer o build ele já será gerado de forma estática, para quando a pessoa clicar ele abrir imediatamente, os outros paths vão sendo gerados conforme a pessoa vai abrindo novas páginas.

```javaScript
export const getStaticPaths: GetStaticPaths = async () => {
  const { data } = await api.get('episodes', {
    params: {
      _limit: 2,
      _sort: 'published_at',
      _order: 'desc'
    }
  });

  const path = data.map((episode) => {
    return {
      params: {
        slug: episode.id
      }
    }
  })

  return {
    paths: path,
    fallback: 'blocking'
  }
}
```

* Quando o fallback: false: - Se o usuário clicar em um episódio ele será redirecionado para o erro 404 pois nenhum episódio foi criado de forma estática no momento da build

```javaScript
export const getStaticPaths: GetStaticPaths = async () => {
  return {
    paths: [],
    fallback: false, 
  }
}
```

* Quando o fallback: true - o método getStaticProps é executado pelo lado do client (browser), e ele demora um pouco para executar então os dados estarão vazios, pois a requisição do getStaticPaths demora um pouco para acontecer
```javaScript
export const getStaticPaths: GetStaticPaths = async () => {
  return {
    paths: [],
    fallback: true, 
  }
}
```
Para resolver o erro pela demora:

```javaScript
import { useRouter } from 'next/router';

export default function Episode({episode}: EpisodeProps) {
  //const router = useRouter();
  //Para pegar a rota router.query.slug

  const router = useRouter();

  if(router.isFallback) {
    return <p>Carregando</p>
  }
}
```

### 8 Lib rc-slider , barra de reprodução do podcast

```sh
yarn add rc-slider
```

_src/components/Player/index.jsx_
```javaScript
import Slider from 'rc-slider';
import 'rc-slider/assets/index.css';

<Slider
  trackStyle={{backgroundColor: '#04d361'}}
  railStyle={{ backgroundColor: '#9f75ff'}}
  handleStyle={{ borderBlockColor: '#04d361', borderWidth: 4}}
/>
```

### 9 Manipular elementos do html utilizando useRef

Aqui estamos recuperando a referência da tag 'audio':

* useRef: sempre inicia com null;

```javaScript
import { useContext, useEffect, useRef } from 'react';

export default function Player() {
  const audioRef = useRef<HTMLAudioElement>(null);
  const [progress, setProgress] = useState(0);

  const {
    episodeList,
    currentEpisodeIndex,
    isPlaying,
    isLooping,
    toggleLoop,
    togglePlay,
    toggleShuffle,
    isShuffling,
    setPlayingState,
    playNext,
    playPrevious,
    hasPrevious,
    hasNext,
    clearPlayerState
  } = usePlayer();

  useEffect(() => {
    if (!audioRef.current) return;

    if (isPlaying) {
      audioRef.current.play();
    } else {
      audioRef.current.pause();
    }
  }, [isPlaying]);

  function setupProgressListener() {
    audioRef.current.currentTime = 0;

    audioRef.current.addEventListener('timeupdate', () => {
      setProgress(Math.floor(audioRef.current.currentTime));
    });
  }

  ...
   { episode && (
      <audio
        src={episode.url}
        ref={audioRef}
        loop={isLooping}
        autoPlay
        onEnded={handleEpisodeEnded}
        onPlay={() => setPlayingState(true)}
        onPause={() => setPlayingState(false)}
        onLoadedMetadata={setupProgressListener}
      />
    )}
  ...
}
```

Missões:
Deixar site responsivo;
Transformar em pwa;
tema Dark;
Electron;



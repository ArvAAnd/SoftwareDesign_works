# Отчёт по реализации: Companies & Vacancies

Дата: 2025-11-24

**Краткое описание реализованного функционала**

- **Ресурсы**: реализованы CRUD для `companies` и `vacancies`.
- **Маршруты**: файл-ориентированные маршруты TanStack Router: `/companies`, `/companies/new`, `/companies/$companyId`, `/vacancies`, `/vacancies/new`, `/vacancies/$vacancyId`.

**Приклади ключового коду**
- Xуки для TanStack Query
```ts
const getVacancies = async (): Promise<Vacancy[]> => {
  const res = await apiClient.get('/vacancies');
  const payload = res.data;
  if (Array.isArray(payload)) return payload as Vacancy[];
  if (payload && Array.isArray(payload.data)) return payload.data as Vacancy[];
  return [];
};
export const useVacancies = () => useQuery<Vacancy[]>({ queryKey: ['vacancies'], queryFn: getVacancies });

const getVacancyById = async (id: number): Promise<Vacancy> => {
  const res = await apiClient.get(`/vacancies/${id}`);
  const payload = res.data;
  if (payload && payload.data) {
    if (Array.isArray(payload.data)) return payload.data[0];
    return payload.data as Vacancy;
  }
  return payload as Vacancy;
};
export const useVacancy = (id: number) => useQuery<Vacancy>({ queryKey: ['vacancies', id], queryFn: () => getVacancyById(id) });


const createVacancy = async (payload: Omit<Vacancy, 'id'>): Promise<Vacancy> => {
  const params = new URLSearchParams();
  params.append('title', payload.title);
  if (payload.description) params.append('description', payload.description);
  if (payload.salary != null) params.append('salary', String(payload.salary));
  if (payload.company?.id) params.append('employer_id', String(payload.company.id));
  if ((payload as any).companyId) params.append('companyId', String((payload as any).companyId));
  const res = await apiClient.post('/vacancies', params.toString(), { headers: { 'Content-Type': 'application/x-www-form-urlencoded' }});
  return res.data;
};
export const useCreateVacancy = () => {
  const queryClient = useQueryClient();
  return useMutation({ mutationFn: createVacancy, onSuccess: () => queryClient.invalidateQueries({ queryKey: ['vacancies'] }) });
};
```

- схема Zod
```ts
const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(2, 'Password too short'),
});
```

**Скриншоты роботы программы**
- Страница компаний
![companies page](images/companiesPage.png)

- Вкладка Network у DevTools
![network](images/network.png)

- Страница входа
![login](images/login.png)

- Ошибка валидации Zod
![zod](images/zod.png)

**Комментарии по особенностям реализации и проблемам, с которыми столкнулись**

- Порядок хуков

  Были случаи раннего `return` (например, `if (isLoading) return <div>Loading...</div>`) до вызова некоторых хуков в `VacancyEditPage.tsx`. Это вызвало ошибку React о смене порядка хуков. Ошибка устранена: все хуки вызываются последовательно, возвраты выполняются после них.

- Формат отправки (x-www-form-urlencoded)
  
  Для совместимости с бэкендом (Postman-примеры) create/update формируются через `URLSearchParams` и отправляются с заголовком `application/x-www-form-urlencoded`. Если бекенд ожидает JSON, потребуется изменить контракт. Сейчас в коде есть двоичная поддержка полей `employer_id` и `companyId` — это подстраховка.

- Нормализация ответов
  
  Бэкенд возвращает как "сырой" массив, так и конверты `{ message, data }` или `{ rows: [...] }`. В хуках есть нормализация (`payload.data`, `payload.rows`, `payload.payload`), чтобы UI надёжно работал независимо от формы ответа.

- Роутинг
  
  Проект использует сгенерированный `routeTree.gen.ts` от TanStack Router. При добавлении/переименовании файлов маршрутов нужно регенерировать дерево маршрутов.




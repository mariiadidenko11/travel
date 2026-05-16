# БезМеж 
# Auth: <token>
# Timestamps: ISO 8601 (UTC)
# IDs: UUID v4
# Soft-delete: all DELETE endpoints move records to deleted_at state


# 1 Authentification
# Covers: login, register, forgot-password pages

POST    /auth/register
  # Створення нового користувача
  # Body: { name, email, password }
  # Returns: { user, access_token, refresh_token }

POST    /auth/login
  # Автентифікувати за поштою та паролем
  # Body: { email, password, remember_me? }
  # Returns: { user, access_token, refresh_token }

POST    /auth/logout
  # Анулювати поточний сеанс /токен оновлення
  # Body: { refresh_token }
  # Returns: 204 No Content

POST    /auth/refresh
  # Body: { refresh_token }
  # Returns: { access_token, refresh_token }

POST    /auth/forgot-password
  # Send a password-reset link to the user's email.
  # Body: { email }
  # Returns: 200 { message }

POST    /auth/reset-password ?
  # Надіслати посилання для скидання пароля на пошту?
  # Body: { token, password, password_confirmation }
  # Returns: 200 { message }

GET     /auth/me
  # Повертає профіль авторизованого користувача
  # Returns: { user }

PATCH   /auth/me
  # Оновлює профіль авторизованого користувача (ім’я)
  # Body: { name }
  # Returns: { user }

DELETE  /auth/me
  # Видалення акку користувача (GDPR)
  # Returns: 204 No Content


# 2 Подорожі
# Covers: all-trips, add-trip, overview pages

GET     /trips
  # Усі поїздки
  # Query: ?status=all|active|planned|completed&sort=date_start&page=1&limit=20
  # Returns: { data: [trip], meta: { total, page, limit } }

POST    /trips
  # Додати нову подорож
  # Body: { name, destination, date_start, date_end, budget, currency,
  #          cover_url?, vibe?, notes? }
  # Returns: { trip }

GET     /trips/:tripId
  # Отримати інфу про одну поїздку
  # Returns: { trip, stats: {  tasks_total, tasks_done, bookings_total,photos_total, notes_total, budget_spent, budget_total, budget_remaining, budget_usage_percent, completion_rate, task_completion_rate, total_days, days_passed, days_left , overall_progress_percent} }

PATCH   /trips/:tripId
  # Оновити деталі поїздки
  # Body: { name?, destination?, date_start?, date_end?, budget?,
  #          currency?, cover_url?, status?, vibe?, notes? rating?}
  # Returns: { trip }

DELETE  /trips/:tripId
  # Видалення подорожі
  # Returns: 204 No Content

POST    /trips/:tripId/restore
  # Відновлення м’яко видаленої поїздки.
  # Returns: { trip }

GET     /trips/stats/summary
  # Статистика
  # Returns: { total_trips, total_days, total_countries, total_places_visited, total_spent_by_currency,spending_by_category  countries_breakdown, avg_trip_duration_days, longest_trip, shortest_trip, completion_rates_by_trip, top_country_by_spending, top_country_by_places, most_frequent_destination, most_active_trip, bookings_by_category }



# 3 — Завдання (To-Do)
# Covers: to-do page

GET     /trips/:tripId/tasks
  # Перелічіть усі завдання для поїздки
  # Query: ?status=all|active|done&sort=created_at
  # Returns: { data: [task] }

POST    /trips/:tripId/tasks
  # Нове завдання
  # Body: { text, vibe?, time_from?, time_to? }
  # Returns: { task }

PATCH   /trips/:tripId/tasks/:taskId
  # Оновіть завдання
  # Body: { text?, done?, vibe?, time_from?, time_to? }
  # Returns: { task }

DELETE  /trips/:tripId/tasks/:taskId
  # Видалення завдання
  # Returns: 204 No Content

POST    /trips/:tripId/tasks/:taskId/restore
  # Відновлення видаленого завдання
  # Returns: { task }



#  4 — Місця
# Covers: places page

GET     /trips/:tripId/places
  # Усі місця
  # Query: ?category=beach|landmark|restaurant|viewpoint|nature|
  #                  entertainment|shopping|other
  #         &status=planned|visited|favourite?
  # Returns: { data: [place] }

POST    /trips/:tripId/places
  # Додати нове місце
  # Body: { name, category, time_from?, time_to?, note? }
  # Returns: { place }

PATCH   /trips/:tripId/places/:placeId
  # Редагувати місце
  # Body: { name?, category?, status?, visited?, favourite?,
  #          time_from?, time_to?, note? }
  # Returns: { place }

DELETE  /trips/:tripId/places/:placeId
  # Видалити місце
  # Returns: 204 No Content

POST    /trips/:tripId/places/:placeId/restore
  # Відновити
  # Returns: { place }

PATCH   /trips/:tripId/places/:placeId/visited
  # Перемикання статусу відвідування прапорцем
  # Body: { visited: boolean }
  # Returns: { place }

PATCH   /trips/:tripId/places/:placeId/favourite ?
  # Улюблені
  # Body: { favourite: boolean }
  # Returns: { place }



# 5 — Бронювання
# Covers: booking page
# Categories: accommodation | transport | food | leisure |  shopping | other

GET     /trips/:tripId/bookings
  # Усі бронювання для поїздки
  # Query: ?category=accommodation|transport|food|leisure|
  #                  shopping|park|beach|other
  # Returns: { data: [booking] }

POST    /trips/:tripId/bookings
  # Нове бронювання
  # Body: { category, title, ...category_specific_fields,
  #          price?, currency?, notes? }
  # Returns: { booking }
  # проживання → { location, type, check_in, check_out,
  #                   guests, room_type, reference, notes }
  # транспорт     → { from, to, departure_at, return_at,
  #                   transport_type, passengers, pickup_details, notesr }
  # харчування          → { venue, location, reserved_at, party_size? notes }
  # активнітсь\ дозвілля       → { activity, location, scheduled_at, duration_minutes,
  #                   tickets, leisure_type, notes }
  # шопінг      → { store, location, planned_at, purpose, budget_estimate, notes }

  # інше         → { custom_title, location, scheduled_at,
  #                   description, notes }

PATCH   /trips/:tripId/bookings/:bookingId
  # Оновити\Редагувтаи
  # Body: { ...any booking fields }
  # Returns: { booking }

DELETE  /trips/:tripId/bookings/:bookingId
  # Видалити
  # Returns: 204 No Content

POST    /trips/:tripId/bookings/:bookingId/restore
  # Повернути
  # Returns: { booking }



# 6 — Нотатки
# Covers: notes page

GET     /trips/:tripId/notes
  # Усі нотатки

POST    /trips/:tripId/notes
  # Додати
  # Body: { title, body? }
  # Returns: { note }

PATCH   /trips/:tripId/notes/:noteId
  # Редагувати
  # Body: { title?, body? }
  # Returns: { note }

DELETE  /trips/:tripId/notes/:noteId
  # Видалити
  # Returns: 204 No Content

POST    /trips/:tripId/notes/:noteId/restore
  # ПОвернути
  # Returns: { note }



# 7 — Галерея
# Covers: gallery page

GET     /trips/:tripId/photos
  # Усі фото з подорожі
  # Query: ?category=landscape|architecture|food|beach|people|evening|other &favourite=true
  # Returns: { data: [photo] }

POST    /trips/:tripId/photos
  # Завантажте одну або декілька фотографій
  # Body: multipart — files[] (image/png),
  #        category?, title?
  # Returns: { data: [photo] }  (one entry per uploaded file)

PATCH   /trips/:tripId/photos/:photoId
  # Редагувати (назва, категорія)
  # Body: { title?, category? }
  # Returns: { photo }

PATCH   /trips/:tripId/photos/:photoId/favourite
  # Улюблені
  # Body: { favourite: boolean }
  # Returns: { photo }

DELETE  /trips/:tripId/photos/:photoId
  # Видалення
  # Returns: 204 No Content

POST    /trips/:tripId/photos/:photoId/restore
  # ПОвернути
  # Returns: { photo }


# MODULE 8 — STATISTICS
# Covers: statistics page (all computed server-side)

GET     /stats/summary
  # Глобальні картки - країна з найбільшими витратами, найпопулярніша за відвідуваннями, найвідвідуваніший напрямок, найдорожча поїздка, найактивніша поїздка, загальна кількість днів
  # Query: ?period=all|last_year|last_6_months
  # Returns: { highest_spending_country, most_explored_country,
  #             most_frequent_destination, premium_trip,
  #             most_active_trip, total_travel_days }

GET     /stats/budget
  # Аналіз бюджету, витрати на поїздку
  # Query: ?period=all|last_year|last_6_months
  # Returns: { data: [{ trip_id, trip_name, total_spent, currency }],
  #             min_trip, max_trip }

GET     /stats/activity
  # Огляд активності, заплановані та відвідані місця за поїздку
  # Query: ?period=all|last_year|last_6_months
  # Returns: { data: [{ trip_id, trip_name, planned, visited,
  #                      completion_rate }], top_performer }

GET     /stats/countries 
  # Аналітика по країнах 
  # Query: ?period=all|last_year|last_6_months
  # Returns: { data: [{ country, total_spent, places_count,
  #                      days_spent, trips_count }] }


GET     /stats/efficiency
  # Оцінки ефективності за поїздку: виконання завдання + відвіданні місця 
  # Query: ?period=all|last_year|last_6_months
  # Returns: { data: [{ trip_id, trip_name, score, tasks_done_pct,
  #                      places_visited_pct }],
  #             avg_score, consistency }

GET     /stats/time
  # Час подорожі: загальна кількість днів, найдовша поїздка, найкоротша, середня
  # Query: ?period=all|last_year|last_6_months
  # Returns: { total_days, longest_trip, shortest_trip,
  #             avg_duration_days, sparkline: [{ date, days }] }

GET     /stats/bookings ?
  # Огляд бронювань
  # Query: ?period=all|last_year|last_6_months
  # Returns: { accommodation, transport, food, leisure,
  #             most_used_category }



# 9 — USER PROFILE, SETTINGS

GET     /users/me
  # Профіль користувача (name, email, avatar?, joined_at?).
  # Returns: { user }

PATCH   /users/me  
  # Редагувати
  # Body: { name, avatar_url, }
  # Returns: { user }

PATCH   /users/me/password
  # Зміна паролю
  # Body: { current_password, new_password, new_password_confirmation }
  # Returns: 200 { message }


DELETE  /users/me
  # ВИдалення
  # Returns: 200 { scheduled_deletion_at }



# 10 — FILE UPLOADS 
# Універсальний ендпоінт для завантаження файлів (для галереї та обкладинки поїздки)
# Storage: S3-compatible (presigned URL pattern).

POST    /uploads/presign
  # Запитує тимчасове посилання для прямого завантаження у S3, але для великих файлів
  # Body: { filename, content_type, size_bytes, context: 'photo'|'cover' }
  # Returns: { upload_url, file_url, expires_at }

DELETE  /uploads
  # Видалити за посиланням
  # Body: { file_url }
  # Returns: 204 No Content



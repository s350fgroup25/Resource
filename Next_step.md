
## Week data ('20260701' AND '20260707')
- user_whole_journey_table_week (1993893 lines)
<img width="1920" height="911" alt="618556575-250777ba-583c-4635-bfae-e9ed76c2d799" src="https://github.com/user-attachments/assets/843807eb-0ef7-4971-a138-02b56d24e082" />
<img width="1920" height="957" alt="618560259-495e9b67-1da0-4500-8a34-c2f2b74cf362" src="https://github.com/user-attachments/assets/3eb19e25-4789-4ef6-b88c-2f873cf9f5a5" />
<img width="1920" height="911" alt="618556665-3b54b016-239e-449e-b420-82f139b1d945" src="https://github.com/user-attachments/assets/9049d1ad-0555-4eae-8c5c-554b93a63aac" />

- draft_title_table_week (1237 line)
<img width="1920" height="911" alt="618557455-8be6b5b4-953f-4987-9527-01b7283e69be" src="https://github.com/user-attachments/assets/7d5e18a2-f4ee-4513-b9fd-f182d3dcb909" />

- draft_theme_table_week (60521 lines) 
<img width="1920" height="911" alt="618558144-43b7a847-c9a8-4e38-a161-122a27b69c74" src="https://github.com/user-attachments/assets/bfab7a0d-806f-4568-b217-72eb59ae1cda" />

- unique_user_count (feature table only 8152 lines )
<img width="866" height="333" alt="618559247-ec083817-0b13-4656-947f-97ef0d2321dd" src="https://github.com/user-attachments/assets/1e3edd0a-c482-48ce-a9ae-7b69ac1f9d82" />


## month data ('20260601' AND '20260630')
- user_whole_journey_table_month (1065584 lines)
<img width="841" height="348" alt="618559924-87eb2390-1322-4d5c-9ef8-1b39c5038bdc" src="https://github.com/user-attachments/assets/87272510-e61b-4d40-b115-a4f1201de202" />
<img width="1920" height="957" alt="618560259-495e9b67-1da0-4500-8a34-c2f2b74cf362" src="https://github.com/user-attachments/assets/422c8395-c4c5-482c-8ff3-483f75518f34" />

- draft_title_table_month (935 line)
<img width="1920" height="911" alt="618561908-c74c7eb6-bdcf-4059-b6fa-a7f201a756e2" src="https://github.com/user-attachments/assets/4de03e52-eb11-445b-a3f6-23224155ec2d" />

- draft_theme_table_month (42980 lines)
<img width="1920" height="911" alt="618561908-c74c7eb6-bdcf-4059-b6fa-a7f201a756e2" src="https://github.com/user-attachments/assets/7907cf62-5058-46d6-8f22-f91c598651f7" />
- unique_user_count (feature table only 5083 lines )
<img width="1920" height="911" alt="618561372-a14a7abf-8e5d-4392-be35-eb980d1e658a" src="https://github.com/user-attachments/assets/7a03233d-0ffe-404a-af2b-12c79f234cdf" />



## Feature extraction (by day (events_20260705) )

### 1. user whole journey table :
- hong-kong-project.analytics_332614377.user_whole_journey_table
- The date is 20260705, with a total of 68240 rows, and is only members 
<img width="1920" height="911" alt="617469723-824ad26e-b78e-4b51-9abf-04239e2c7193" src="https://github.com/user-attachments/assets/e9311046-e4b0-4201-8117-77fa42558feb" />


      CREATE OR REPLACE TABLE `hong-kong-project.analytics_332614377.user_whole_journey_table` AS
      WITH users_with_id AS (
        -- Find all user_pseudo_id that have at least one user_id
        SELECT DISTINCT
          user_pseudo_id,
          -- Get the user_id associated with this pseudo_id
          FIRST_VALUE(user_id IGNORE NULLS) OVER (
            PARTITION BY user_pseudo_id 
            ORDER BY event_timestamp
          ) AS associated_user_id
        FROM `hong-kong-project.analytics_332614377.events_20260705`
        WHERE user_id IS NOT NULL
      ),
      user_journey AS (
        SELECT
          e.user_pseudo_id,
          -- Show NULL if this specific event has no user_id, even if user eventually logs in
          e.user_id AS user_id,
          e.event_name AS en,
          (SELECT value.string_value FROM UNNEST(e.event_params) WHERE key = 'page_location') AS dl,
          (SELECT value.string_value FROM UNNEST(e.event_params) WHERE key = 'page_title') AS dt,
          (SELECT value.int_value FROM UNNEST(e.event_params) WHERE key = 'ga_session_id') AS session_time,
          e.event_timestamp
        FROM `hong-kong-project.analytics_332614377.events_20260705` e
        INNER JOIN users_with_id u ON e.user_pseudo_id = u.user_pseudo_id
        WHERE (SELECT value.string_value FROM UNNEST(e.event_params) WHERE key = 'page_location') IS NOT NULL
      )
      SELECT *
      FROM user_journey
      ORDER BY user_pseudo_id, session_time, event_timestamp;

### 2. Event name :
<img width="1920" height="911" alt="617470491-f15b9dfc-4219-454a-b70d-fa9612d5e1fe" src="https://github.com/user-attachments/assets/c74fccb4-eab1-4a97-b435-d1109e8134f0" />

  
            WITH event_stats AS (
            SELECT
            en,
            COUNT(*) AS event_count
            FROM `hong-kong-project.analytics_332614377.user_whole_journey_table`
            GROUP BY en
            )
            SELECT
            (SELECT COUNT(DISTINCT en) FROM `hong-kong-project.analytics_332614377.user_whole_journey_table`) AS total_distinct_events,
            STRING_AGG(DISTINCT en, ', ' ORDER BY en) AS all_event_names,
            (SELECT COUNT(*) FROM `hong-kong-project.analytics_332614377.user_whole_journey_table`) AS total_events
            FROM `hong-kong-project.analytics_332614377.user_whole_journey_table`;


### 3. draft table : theme = 'null' 

3.1 draft theme table: 
- have dl (page location (URL)), dt (page title) , theme
- 6605 lines

<img width="1920" height="911" alt="617910630-a530ae20-e017-4b1b-9656-edb6097a7c1b" src="https://github.com/user-attachments/assets/ac21fec3-e949-4ec9-9380-79d84efaf13f" />
<img width="1920" height="947" alt="617974145-d7f488da-0b47-487f-a660-512b3da492ef" src="https://github.com/user-attachments/assets/a425d893-7e71-4d95-8e81-5d00d4b20807" />

- hong-kong-project.analytics_332614377.draft_theme_table
  
        CREATE OR REPLACE TABLE `hong-kong-project.analytics_332614377.draft_theme_table` AS
        SELECT DISTINCT
          dl,
          dt,
          CAST(NULL AS STRING) AS theme
        FROM `hong-kong-project.analytics_332614377.user_whole_journey_table`
        WHERE dl IS NOT NULL 
          AND dt IS NOT NULL
        ORDER BY dl, dt;

3.2 draft_title_table:
- just dt (page title) , theme
- 260 lines
<img width="1920" height="947" alt="617974956-87a4c400-ed1f-4b14-b5e8-208c6e7f6d10" src="https://github.com/user-attachments/assets/11629767-3cdb-459c-9016-e37146f2f6d4" />


- hong-kong-project.analytics_332614377.draft_title_table

      CREATE OR REPLACE TABLE `hong-kong-project.analytics_332614377.draft_title_table` AS
      SELECT DISTINCT
        dt,
        CAST(NULL AS STRING) AS theme
      FROM `hong-kong-project.analytics_332614377.user_whole_journey_table`
      WHERE dt IS NOT NULL
      ORDER BY dt;


### 4. Gemini table

4.1 draft_title_table_with_ai
- just use 260 lines  , can cover all user_whole_journey_table  , less the gemini usage
<img width="1920" height="947" alt="617981013-c357ff01-d52d-44f2-960e-4d57ba651309" src="https://github.com/user-attachments/assets/1f055ff8-c117-4785-bf3b-31ac5713c58f" />
<img width="1920" height="947" alt="617982036-9ca5c05f-fc49-4c5c-a50c-902a750a1c14" src="https://github.com/user-attachments/assets/a859cf32-dffe-45ac-aeba-750767772713" />
<img width="958" height="910" alt="617985984-7e7ecfa4-86b7-444b-9bbc-fbfb56c7dc75" src="https://github.com/user-attachments/assets/f7532c98-dd84-49ed-9900-704af6214674" />


      -- For Gemini in BigQuery - Classify page titles only
      CREATE OR REPLACE TABLE `hong-kong-project.analytics_332614377.draft_title_table_with_ai` AS
      SELECT
        dt,
        ML.GENERATE_TEXT(
          MODEL `hong-kong-project.analytics_332614377.gemini_model`,
          CONCAT(
            'You are a page categorization expert. Your task is to classify web pages into the most appropriate category from the list below.
      
      **Categories:**
      [Sport & Action, Travel & Adventure, Food & Wine, Music & Art, Health, Wealth, Lifestyle, Archived, Latest Promotion, Miffy Special Offer, Featured Offers, Happy Wednesday, Bidding, Travel & Adventure, Health & Wellness, Electronic & Gadgets, Home & Kitchen, New Arrival, Top 10 Rewards, Parenting, Medical, Accident, Housing, Life, Events & Tickets, Account & Login, Home Page, Wallet Page, Profile Page, 任務卡, 賺分貼士, Invitation Page, Info Page, FAQ Page, Content Engagement, Promotion/Cross Sell, Shopping/Conversion, Other / Undefined]
      
      **Instructions:**
      1. Analyze the page title provided below
      2. Choose ONE category that best describes the page content
      3. Return ONLY the category name - no explanations, no extra text
      
      **Page Title:** ',
            dt
          )
        ) AS theme
      FROM `hong-kong-project.analytics_332614377.draft_title_table`
      WHERE theme IS NULL;


4.2 draft theme table with ai:
- need 6605 lines then cover all user_whole_journey_table
- add URL as supplementary information can make the result more accurate , but its requires frequent revisions
<img width="1920" height="911" alt="617910339-e4ab820c-8882-4d2b-b084-cea1566d37a2" src="https://github.com/user-attachments/assets/c3c22f0c-3ded-4fe5-946f-c5e3178731bf" />

      -- For Gemini in BigQuery
      CREATE OR REPLACE TABLE `hong-kong-project.analytics_332614377.draft_theme_table_with_ai` AS
      SELECT
        dl,
        dt,
        ML.GENERATE_TEXT(
          MODEL `your_model_name`,
          CONCAT(
            'Classify this page into one of these themes: ',
            'Sport & Action, Travel & Adventure, Travel Insurance, Food & Wine, ',
            'Art & Entertainment, Health & Wellness, Wealth & Investment, ',
            'Lifestyle & Daily Life, Archived, Latest Promotion, Miffy Special Offer, ',
            'Featured Offers, Happy Wednesday, Bidding, Electronic & Gadgets, ',
            'Home & Kitchen, New Arrival, Top 10 Rewards, Parenting, Accident, ',
            'Housing, Life, Four-in-One Insurance, Events & Tickets, Account & Login, ',
            'Home Page, Wallet Page, Profile Page, 任務卡, 賺分貼士, Invitation Page, ',
            'Info Page, FAQ Page, Content Engagement, Promotion/Cross Sell, ',
            'Shopping/Conversion, Other / Undefined',
            '. Page URL: ', dl, '. Page Title: ', dt,
            '. Return only the theme name.'
          )
        ) AS theme
      FROM `hong-kong-project.analytics_332614377.draft_theme_table`
      WHERE theme IS NULL;

## Here is an example of how to use BigQuery tables in Gemini.
- ask for gemini 
<img width="1919" height="1055" alt="617473758-b25b4eb2-1297-41d5-87ee-d7f432286eb0" src="https://github.com/user-attachments/assets/ce2759f0-5677-46a7-a0f0-fb989007d763" />

- Gemini table : 900 line now , total in 5/7/2026 are 6605
<img width="1916" height="1002" alt="617468065-5ba6c58d-2805-4f9c-8d83-984e733f87a0" src="https://github.com/user-attachments/assets/df47417b-a013-4f2e-8ca6-c3856fbefb44" />

## Dynamic Categories with a Categories Table
Step 1: Create the Categories Table

      -- Create the categories master table with correct data types
      CREATE OR REPLACE TABLE `hong-kong-project.analytics_332614377.categories_master` (
        category_id INT64,  -- Changed from STRING to INT64
        category_name STRING,
        is_active BOOLEAN,
        created_at TIMESTAMP,
        updated_at TIMESTAMP,
        sort_order INT64
      );
      
      -- Insert all your categories with proper IDs
      INSERT INTO `hong-kong-project.analytics_332614377.categories_master`
      SELECT 
        ROW_NUMBER() OVER (ORDER BY category_name) AS category_id,  -- Auto-increment ID
        category_name,
        TRUE AS is_active,
        CURRENT_TIMESTAMP() AS created_at,
        CURRENT_TIMESTAMP() AS updated_at,
        ROW_NUMBER() OVER (ORDER BY category_name) AS sort_order
      FROM UNNEST([
        'Sport & Action',
        'Travel & Adventure',
        'Food & Wine',
        'Music & Art',
        'Health',
        'Wealth',
        'Lifestyle',
        'Archived',
        'Latest Promotion',
        'Miffy Special Offer',
        'Featured Offers',
        'Happy Wednesday',
        'Bidding',
        'Health & Wellness',
        'Electronic & Gadgets',
        'Home & Kitchen',
        'New Arrival',
        'Top 10 Rewards',
        'Parenting',
        'Medical',
        'Accident',
        'Housing',
        'Life',
        'Events & Tickets',
        'Account & Login',
        'Home Page',
        'Wallet Page',
        'Profile Page',
        '任務卡',
        '賺分貼士',
        'Invitation Page',
        'Info Page',
        'FAQ Page',
        'Content Engagement',
        'Promotion/Cross Sell',
        'Shopping/Conversion',
        'Other / Undefined'
      ]) AS category_name
      -- Remove duplicates by using DISTINCT
      QUALIFY ROW_NUMBER() OVER (PARTITION BY category_name ORDER BY category_name) = 1;

<img width="1920" height="911" alt="618603017-f35cd286-0848-4a7c-b025-2195bd0911f2" src="https://github.com/user-attachments/assets/444089cb-396b-4123-a629-752410bb16cf" />
<img width="1920" height="911" alt="618603203-9161bb5b-c63e-425e-b50f-53d0ab949754" src="https://github.com/user-attachments/assets/b03db2e8-6746-4f50-b100-5983533d960d" />


Step 2: Create a Dynamic Stored Procedure
- **Add a new category** : 

<img width="1108" height="261" alt="618604780-3f98be76-1a1e-4d03-b29b-3881a19f4320" src="https://github.com/user-attachments/assets/3b72ecce-9e20-48c8-b756-18bebad87c5f" />


      -- Add a new category (updated - fixed with dummy FROM)
      CREATE OR REPLACE PROCEDURE `hong-kong-project.analytics_332614377.add_category`(category_name_param STRING)
      BEGIN
        INSERT INTO `hong-kong-project.analytics_332614377.categories_master`
        SELECT 
          (SELECT IFNULL(MAX(category_id), 0) + 1 FROM `hong-kong-project.analytics_332614377.categories_master`),
          category_name_param,
          TRUE,
          CURRENT_TIMESTAMP(),
          CURRENT_TIMESTAMP(),
          (SELECT IFNULL(MAX(sort_order), 0) + 1 FROM `hong-kong-project.analytics_332614377.categories_master`)
        FROM UNNEST([1])  -- Dummy FROM clause
        WHERE NOT EXISTS (
          SELECT 1 
          FROM `hong-kong-project.analytics_332614377.categories_master` 
          WHERE LOWER(category_name) = LOWER(category_name_param)
        );
      END;

- **Deactivate a category** : 

            -- Deactivate a category (soft delete)
            CREATE OR REPLACE PROCEDURE `hong-kong-project.analytics_332614377.deactivate_category`(category_name_param STRING)
            BEGIN
              UPDATE `hong-kong-project.analytics_332614377.categories_master`
              SET 
                is_active = FALSE,
                updated_at = CURRENT_TIMESTAMP()
              WHERE LOWER(category_name) = LOWER(category_name_param);
            END;

- **Reactivate a category** : 

      -- Reactivate a category
      CREATE OR REPLACE PROCEDURE `hong-kong-project.analytics_332614377.reactivate_category`(category_name_param STRING)
      BEGIN
        UPDATE `hong-kong-project.analytics_332614377.categories_master`
        SET 
          is_active = TRUE,
          updated_at = CURRENT_TIMESTAMP()
        WHERE LOWER(category_name) = LOWER(category_name_param);
      END;
  
- **Permanently delete (only if not in use)** :

      -- Permanently delete a category (use with caution)
      CREATE OR REPLACE PROCEDURE `hong-kong-project.analytics_332614377.delete_category_permanent`(category_name_param STRING)
      BEGIN
        DELETE FROM `hong-kong-project.analytics_332614377.categories_master`
        WHERE LOWER(category_name) = LOWER(category_name_param)
        AND category_name NOT IN (
          SELECT DISTINCT theme 
          FROM `hong-kong-project.analytics_332614377.draft_title_table_with_ai`
          WHERE theme IS NOT NULL
        );
      END;
Step 3: Create a Function to Get Categories 
<img width="1444" height="379" alt="618608813-9b437491-44ef-4854-9967-97aef1d6864d" src="https://github.com/user-attachments/assets/c558b9dd-e7da-476b-90c2-64477449340b" />


      CREATE OR REPLACE FUNCTION `hong-kong-project.analytics_332614377.get_category_list`()
      RETURNS STRING
      AS (
        (
          SELECT STRING_AGG('"' || category_name || '"', ', ' ORDER BY sort_order)
          FROM `hong-kong-project.analytics_332614377.categories_master`
          WHERE is_active = TRUE
        )
      );

Step4. 
- Test 1: to see what the function returns
<img width="1399" height="346" alt="618610274-aa783e85-158d-4522-9cb3-02e7d1133a9f" src="https://github.com/user-attachments/assets/9fd63589-d83c-41e8-b0d7-a07c791e7fa5" />


      SELECT 
        CONCAT(
          '**Categories:**\n[',
          `hong-kong-project.analytics_332614377.get_category_list`(),
          ']\n\n'
        ) AS full_prompt_preview;

- Test 2: Add a test category

      CALL `hong-kong-project.analytics_332614377.add_category`('Test Category');

- Test 3: See the category list that will be used in the prompt

      SELECT `hong-kong-project.analytics_332614377.get_category_list`() AS category_list;

<img width="1408" height="405" alt="618611605-2c6debe1-b38b-4e8b-b88b-e408e5eab5b5" src="https://github.com/user-attachments/assets/e7fcf599-736c-45d7-a961-32df6f495e3c" />


- Test 4: Deactivate a category

      CALL `hong-kong-project.analytics_332614377.deactivate_category`('Archived');

- Test 5: Verify deactivation worked

      SELECT * FROM `hong-kong-project.analytics_332614377.categories_master`
      WHERE LOWER(category_name) = LOWER('Test Category')

<img width="1407" height="328" alt="618612442-eba7ad8a-aee3-46ed-ad0c-2c3c3f3ab6a6" src="https://github.com/user-attachments/assets/34e9e73f-eee1-4fe7-b3b8-c40d272e8b94" />


-- Test 6: Permanent delete (use with caution - only if not in use)

      CALL `hong-kong-project.analytics_332614377.delete_category_permanent`('Test Category');

<img width="1409" height="784" alt="618612798-101885b7-cb5b-4670-b894-cd2dc873026d" src="https://github.com/user-attachments/assets/d87dbc18-7b55-44fd-861e-8848a1504472" />


Step 5: Use the Function in Your Main Query

      -- Main classification query
      CREATE OR REPLACE TABLE `hong-kong-project.analytics_332614377.draft_title_table_with_ai` AS
      SELECT
        dt,
        ML.GENERATE_TEXT(
          MODEL `hong-kong-project.analytics_332614377.gemini_model`,
          CONCAT(
            'You are a page categorization expert. Your task is to classify web pages into the most appropriate category from the list below.
      
      **Categories:**
      [',
            `hong-kong-project.analytics_332614377.get_category_list`(),
            ']
      
      **Instructions:**
      1. Analyze the page title provided below
      2. Choose ONE category that best describes the page content
      3. Return ONLY the category name - no explanations, no extra text
      
      **Page Title:** ',
            dt
          )
        ) AS theme,
        CURRENT_TIMESTAMP() AS classified_at
      FROM `hong-kong-project.analytics_332614377.draft_title_table`
      WHERE theme IS NULL;

### 5. user theme journey table
- with :draft_title_table_with_ai
      - Completed, all matches, but the Gemini analysis may contain some errors and is not accurate enough.
<img width="1920" height="947" alt="617983279-58ad1421-21bd-43df-8eb9-11420063e5c7" src="https://github.com/user-attachments/assets/9e0cca81-83af-4ee8-b3f1-2c7b27bb3459" />

- hong-kong-project.analytics_332614377.user_theme_journey_table
  
       CREATE OR REPLACE TABLE `hong-kong-project.analytics_332614377.user_theme_journey_table` AS
            WITH themed_journey AS (
              SELECT
                j.user_id,
                j.user_pseudo_id,
                t.theme,
                j.en,
                j.event_timestamp
              FROM `hong-kong-project.analytics_332614377.user_whole_journey_table` j
              LEFT JOIN `hong-kong-project.analytics_332614377.draft_theme_table_with_ai` t
                ON j.dl = t.dl AND j.dt = t.dt
            )
            SELECT
              user_id,
              theme,
              en,
              ROW_NUMBER() OVER (
                PARTITION BY COALESCE(user_id, user_pseudo_id) 
                ORDER BY event_timestamp
              ) AS visit_order
            FROM themed_journey
            ORDER BY COALESCE(user_id, user_pseudo_id), visit_order;
      
- with :draft_theme_table_with_ai
      - Since draft_title_table_with_ai is not yet complete, there are null values.
<img width="1920" height="911" alt="617908968-f5bc4000-48aa-4f8a-9579-6f1fdbacaf1f" src="https://github.com/user-attachments/assets/e10cce2a-9876-47e0-803b-97119a580491" />
<img width="1920" height="911" alt="617909158-19397331-7837-4118-97b8-6a811d0a5661" src="https://github.com/user-attachments/assets/6d686482-fbbb-40af-b6b4-2495032a02b9" />

- hong-kong-project.analytics_332614377.user_theme_journey_table

      -- Recreate user_theme_journey_table with user_pseudo_id
      CREATE OR REPLACE TABLE `hong-kong-project.analytics_332614377.user_theme_journey_table` AS
      WITH themed_journey AS (
        SELECT
          j.user_id,
          j.user_pseudo_id,  -- Include user_pseudo_id
          t.theme,
          j.en,
          j.event_timestamp
        FROM `hong-kong-project.analytics_332614377.user_whole_journey_table` j
        LEFT JOIN `hong-kong-project.analytics_332614377.draft_theme_table_with_ai` t
          ON j.dl = t.dl AND j.dt = t.dt
      )
      SELECT
        user_id,
        user_pseudo_id,  -- Include in final SELECT
        theme,
        en,
        ROW_NUMBER() OVER (
          PARTITION BY COALESCE(user_id, user_pseudo_id) 
          ORDER BY event_timestamp
        ) AS visit_order
      FROM themed_journey
      ORDER BY COALESCE(user_id, user_pseudo_id), visit_order;

### 6. user_journey_summary
- Create the user_journey_summary table using user_theme_journey_table, which contains all the en and theme columns (dynamic).

<img width="1920" height="911" alt="617910091-2cd8ba39-628b-414b-ab79-7b68b237d308" src="https://github.com/user-attachments/assets/f5560e49-2d93-43ba-af6a-2202e32afce2" />
<img width="1920" height="911" alt="617911278-687fc34a-8e47-4aa0-a015-22024a7da7d0" src="https://github.com/user-attachments/assets/0646c731-896d-49f9-a841-3b862c1706dc" />
<img width="1920" height="911" alt="617911192-2f931a8f-38b1-4023-9b55-9fea5b0be283" src="https://github.com/user-attachments/assets/cb1c34c4-ad21-4250-90fe-167a6bc69495" />
<img width="1920" height="911" alt="618546475-a82104a1-a2c5-43db-b37e-19bc0b7ae4e8" src="https://github.com/user-attachments/assets/4bff51ce-3f62-493d-869f-e699d02256c2" />



      BEGIN
        -- Step 1: DECLARE variables at the very beginning
        DECLARE theme_columns STRING;
        DECLARE event_columns STRING;
        DECLARE all_columns STRING;
      
        -- Step 2: Create temporary table
        CREATE OR REPLACE TEMP TABLE temp_journey_filled AS
        WITH user_mapping AS (
          SELECT DISTINCT
            user_pseudo_id,
            FIRST_VALUE(user_id IGNORE NULLS) OVER (
              PARTITION BY user_pseudo_id 
              ORDER BY visit_order
              ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
            ) AS associated_user_id
          FROM `hong-kong-project.analytics_332614377.user_theme_journey_table`
          WHERE user_id IS NOT NULL
        )
        SELECT
          COALESCE(j.user_id, u.associated_user_id) AS user_id,
          j.user_pseudo_id,
          j.theme,
          j.en,
          j.visit_order
        FROM `hong-kong-project.analytics_332614377.user_theme_journey_table` j
        INNER JOIN user_mapping u ON j.user_pseudo_id = u.user_pseudo_id;
      
        -- Step 3: Create theme list with cleaned names and deduplication
        CREATE OR REPLACE TEMP TABLE theme_list AS
        WITH base AS (
          SELECT 
            theme,
            -- Better cleaning: remove all special characters including parentheses
            REPLACE(
              REPLACE(
                REPLACE(
                  REPLACE(
                    REPLACE(
                      REPLACE(
                        REPLACE(
                          REPLACE(
                            REPLACE(theme, ' ', '_'), 
                          '&', '_'),
                        '/', '_'),
                      '"', ''),
                    "'", ''),
                  '(', ''),
                ')', ''),
              '-', '_'),
            '.', '_') AS clean_theme
          FROM (SELECT DISTINCT theme FROM temp_journey_filled WHERE theme IS NOT NULL)
        ),
        dedup AS (
          SELECT 
            theme,
            clean_theme,
            ROW_NUMBER() OVER (PARTITION BY UPPER(clean_theme) ORDER BY theme) AS dup_count,
            ROW_NUMBER() OVER (ORDER BY theme) AS theme_index
          FROM base
        )
        SELECT 
          theme,
          -- If duplicate, append index
          CASE 
            WHEN dup_count > 1 THEN CONCAT(clean_theme, '_', CAST(dup_count AS STRING))
            ELSE clean_theme
          END AS clean_theme,
          theme_index
        FROM dedup
        ORDER BY theme_index;
      
        -- Step 4: Create event list with cleaned names and deduplication
        CREATE OR REPLACE TEMP TABLE event_list AS
        WITH base AS (
          SELECT 
            en,
            -- Better cleaning: remove all special characters including parentheses
            REPLACE(
              REPLACE(
                REPLACE(
                  REPLACE(
                    REPLACE(
                      REPLACE(
                        REPLACE(
                          REPLACE(
                            REPLACE(
                              REPLACE(en, ' ', '_'), 
                            '&', '_'),
                          '/', '_'),
                        '"', ''),
                      "'", ''),
                    '(', ''),
                  ')', ''),
                '-', '_'),
              '.', '_'),
            ',', '') AS clean_en
          FROM (SELECT DISTINCT en FROM temp_journey_filled)
        ),
        dedup AS (
          SELECT 
            en,
            clean_en,
            ROW_NUMBER() OVER (PARTITION BY UPPER(clean_en) ORDER BY en) AS dup_count,
            ROW_NUMBER() OVER (ORDER BY en) AS event_index
          FROM base
        )
        SELECT 
          en,
          -- If duplicate, append index
          CASE 
            WHEN dup_count > 1 THEN CONCAT(clean_en, '_', CAST(dup_count AS STRING))
            ELSE clean_en
          END AS clean_en,
          event_index
        FROM dedup
        ORDER BY event_index;
      
        -- Step 5: Build theme columns
        SET theme_columns = (
          SELECT STRING_AGG(
            FORMAT("COUNTIF(theme = '%s') AS `%s`", 
                   REPLACE(theme, "'", "\\'"),
                   -- Use the cleaned name, if empty or starts with number, add prefix
                   CASE 
                     WHEN clean_theme = '' OR clean_theme IS NULL THEN CONCAT('theme_', CAST(theme_index AS STRING))
                     WHEN SAFE.SUBSTR(clean_theme, 1, 1) BETWEEN '0' AND '9' THEN CONCAT('theme_', clean_theme)
                     ELSE clean_theme
                   END
            )
            , ', ')
          FROM theme_list
        );
      
        -- Step 6: Build event columns
        SET event_columns = (
          SELECT STRING_AGG(
            FORMAT("COUNTIF(en = '%s') AS `%s`",
                   REPLACE(en, "'", "\\'"),
                   -- Use the cleaned name, if empty or starts with number, add prefix
                   CASE 
                     WHEN clean_en = '' OR clean_en IS NULL THEN CONCAT('event_', CAST(event_index AS STRING))
                     WHEN SAFE.SUBSTR(clean_en, 1, 1) BETWEEN '0' AND '9' THEN CONCAT('event_', clean_en)
                     ELSE clean_en
                   END
            )
            , ', ')
          FROM event_list
        );
      
        -- Step 7: Combine both sets of columns
        SET all_columns = CONCAT(
          IFNULL(theme_columns, ''), 
          IF(theme_columns IS NOT NULL AND event_columns IS NOT NULL, ', ', ''),
          IFNULL(event_columns, '')
        );
      
        -- Step 8: Create the final summary table
        EXECUTE IMMEDIATE FORMAT("""
        CREATE OR REPLACE TABLE `hong-kong-project.analytics_332614377.user_journey_summary` AS
        SELECT
          user_id,
          user_pseudo_id,
          %s
        FROM temp_journey_filled
        WHERE user_id IS NOT NULL
        GROUP BY user_id, user_pseudo_id
        ORDER BY user_id, user_pseudo_id
        """, all_columns);
      
        -- Step 9: Create mapping table for reference
        CREATE OR REPLACE TABLE `hong-kong-project.analytics_332614377.column_mapping` AS
        SELECT 
          'theme' AS column_type,
          theme AS original_name,
          CASE 
            WHEN clean_theme = '' OR clean_theme IS NULL THEN CONCAT('theme_', CAST(theme_index AS STRING))
            WHEN SAFE.SUBSTR(clean_theme, 1, 1) BETWEEN '0' AND '9' THEN CONCAT('theme_', clean_theme)
            ELSE clean_theme
          END AS column_name,
          theme_index AS column_index
        FROM theme_list
        UNION ALL
        SELECT 
          'event' AS column_type,
          en AS original_name,
          CASE 
            WHEN clean_en = '' OR clean_en IS NULL THEN CONCAT('event_', CAST(event_index AS STRING))
            WHEN SAFE.SUBSTR(clean_en, 1, 1) BETWEEN '0' AND '9' THEN CONCAT('event_', clean_en)
            ELSE clean_en
          END AS column_name,
          event_index AS column_index
        FROM event_list
        ORDER BY column_type, column_index;
      END;

### 7. Tag table :
- Find newly added GA tags (including member information)
      - has_travel_coupon : ep.couponTag = 'TRAVEL'
      - has_travel_product : ep.productTag = 'TRAVEL
- hong-kong-project.analytics_332614377.user_tag_labels

      -- Single pass to find users with TRAVEL tags
      CREATE OR REPLACE TABLE `hong-kong-project.analytics_332614377.user_tag_labels` AS
      WITH tag_data AS (
        SELECT
          user_id,
          -- Check if couponTag = 'TRAVEL' exists for this user
          MAX(CASE 
            WHEN ep.key = 'couponTag' AND ep.value.string_value = 'TRAVEL' 
            THEN 1 
            ELSE 0 
          END) AS has_travel_coupon,
          -- Check if productTag = 'TRAVEL' exists for this user
          MAX(CASE 
            WHEN ep.key = 'productTag' AND ep.value.string_value = 'TRAVEL' 
            THEN 1 
            ELSE 0 
          END) AS has_travel_product
        FROM `hong-kong-project.analytics_332614377.events_20260706`,
        UNNEST(event_params) AS ep
        WHERE user_id IS NOT NULL
          AND (ep.key = 'couponTag' OR ep.key = 'productTag')
          AND ep.value.string_value = 'TRAVEL'
        GROUP BY user_id
      )
      SELECT
        user_id,
        has_travel_coupon,
        has_travel_product
      FROM tag_data
      ORDER BY user_id;

- sample result

  | user_id | has_travel_coupon |has_travel_product| 
  |---|---|---|
  | 1001 | 1 | 0|
  | 1002 | 0 | 1| 

### 8. Feature table :
- combine the user_tag_labels with user_journey_summary
- 
      | user_id | user_pseudo_id | theme_Travel | theme_Health | ... | page_view | view_item | add_to_cart | ... | has_travel_coupon | has_travel_product |
      | ------- | -------------- | ------------ | ------------ | --- | --------- | --------- | ----------- | --- | ----------------------- | ------------------------ |
      | 1001    | 12314325       | 1            | 0            | ... | 33        | 5         | 1           | ... | 1                       | 0                        |
      | 1002    | 23445325       | 1            | 0            | ... | 44        | 5         | 1           | ... | 0                       | 1                        |

## Model Training and Evaluation 
1. AutoML in BigQuery ML:
 - Determine Your Goal (AUTOML_REGRESSOR)
       - Classification: Predict a category (e.g., "Will this user make a purchase?")
       - Regression: Predict a numeric value (e.g., "How many times will this user return?") 
- Prepare Your Data
      - label/target column : has_travel_product -- ep.productTag = 'TRAVEL'

       CREATE OR REPLACE MODEL `hong-kong-project.analytics_332614377.user_automl_model`
      OPTIONS(
        model_type='AUTOML_REGRESSOR',  -- or ''AUTOML_CLASSIFIER
        input_label_cols=['has_travel_product']  -- replace with your target column
        budget_hours=1.0
      ) AS
      SELECT *
      FROM `hong-kong-project.analytics_332614377.user_journey_summary`
      WHERE your_label_column IS NOT NULL;  -- only training data with labels

2. Model Evaluation & Prediction
- 2.1 Evaluate the Model

      -- Get training evaluation metrics
      SELECT *
      FROM ML.EVALUATE(
        MODEL `hong-kong-project.analytics_332614377.automl_regressor_model`
      );

- 2.2 Make Predictions

            -- Predict for new users (without labels)
            SELECT *
            FROM ML.PREDICT(
              MODEL `hong-kong-project.analytics_332614377.automl_regressor_model`,
              (
                SELECT 
                  * EXCEPT(user_id, user_pseudo_id)
                FROM `hong-kong-project.analytics_332614377.user_journey_summary`
                WHERE user_id NOT IN (SELECT DISTINCT user_id FROM your_transaction_table)
              )
            );

- 2.3 Export Model Feature Importance
  
      -- Show which features are most important
      SELECT *
      FROM ML.FEATURE_INFO(
        MODEL `hong-kong-project.analytics_332614377.automl_regressor_model`
      );

## Prerequisites & Permissions

Step 1: Prerequisites & Permissions 

- Required IAM Roles :

| Role Name | Description |
|-----------|-------------|
| `bigquery.connectionAdmin` | Full control over BigQuery connections |
| `bigquery.models.create` | Create new models in datasets |
| `bigquery.models.getData` | Read model data and metadata |
| `bigquery.models.updateData` | Update model training data |
| `bigquery.dataEditor` | Read, write, and manage table data |
| `bigquery.jobUser` | Run BigQuery jobs (queries, load, export) |
| `bigquery.tables.getData` | Read table data |
| `bigquery.tables.updateData` | Update table data |
| `aiplatform.user` | Use Vertex AI services (including Gemini) |

- Recommended Role Assignments

| Principal | Recommended Roles | Why |
|-----------|-------------------|-----|
| **Your User Account** | • `roles/bigquery.admin` (or `dataEditor` + `jobUser`)<br>• `roles/aiplatform.user` | To create connections, models, and run all SQL |
| **BigQuery Connection Service Account** | • `roles/aiplatform.user` | Required for `ML.GENERATE_TEXT` to call Gemini |
| **Automation Service Account**  | • `roles/bigquery.dataEditor`<br>• `roles/bigquery.jobUser` | For scheduled jobs and automated workflows |


- Enable Required APIs

| API | Link | Purpose |
|-----|------|---------|
| **BigQuery API** | [Enable BigQuery API](https://console.cloud.google.com/apis/library/bigquery.googleapis.com) | Core BigQuery service |
| **BigQuery Connection API** | [Enable BigQuery Connection API](https://console.cloud.google.com/apis/library/bigqueryconnection.googleapis.com) | Required for connections to Vertex AI |
| **Vertex AI API** | [Enable Vertex AI API](https://console.cloud.google.com/apis/library/aiplatform.googleapis.com) | Required for Gemini and AI services |

Step 2: Create Gemini Remote Model (for Theme Classification)
- 2.1 Create a BigQuery Connection

      -- Create a connection to Vertex AI
      CREATE OR REPLACE CONNECTION `hong-kong-project.analytics_332614377.vertex_ai_connection`
      AS CONNECTION TYPE `CLOUD_RESOURCE`
      OPTIONS (
        location = 'us-central1'  -- Choose your region
      );

- 2.2 Grant Permissions to the Connection
      - After running the above SQL, find the Service Account ID generated for your connection:

      -- Find the service account
      SELECT service_account_id
      FROM `hong-kong-project.analytics_332614377.INFORMATION_SCHEMA.CONNECTIONS`
      WHERE connection_name = 'vertex_ai_connection';

- 2.3 Create the Gemini Remote Model

      -- Create remote model pointing to Gemini
      CREATE OR REPLACE MODEL `hong-kong-project.analytics_332614377.gemini_model`
      REMOTE WITH CONNECTION `hong-kong-project.analytics_332614377.vertex_ai_connection`
      OPTIONS (
        ENDPOINT = 'gemini-2.5-flash-lite'  -- or 'gemini-pro'
      );

<img width="890" height="305" alt="617918153-08fd7fce-5410-44d9-b4fe-d3cbd0b648a7" src="https://github.com/user-attachments/assets/2e93901b-7e5f-48cb-a4da-fed95ab8953c" />


## Useful Links
- ML Model  : https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/bigqueryml-syntax-create
<img width="1920" height="911" alt="618537838-5eca0b59-407a-461b-902d-cc99fb4d3713" src="https://github.com/user-attachments/assets/b9b57a04-6b8d-47f6-987b-8c733056cdbf" />

- BigQuery Model type 
<img width="1920" height="911" alt="618538431-2fce60e5-e54d-474d-b25d-4bd3915c6260" src="https://github.com/user-attachments/assets/bb1ded0c-7029-44ed-8225-8535f8168dd5" />
<img width="1920" height="911" alt="618538579-f4a6c9ee-0aa6-49e4-b364-74724374c6f1" src="https://github.com/user-attachments/assets/f2c031c8-e100-4c81-8d9e-3f4f163a852c" />


- AutoML  : https://docs.cloud.google.com/bigquery/docs/reference/standard-sql/bigqueryml-syntax-create-automl
 <img width="958" height="910" alt="617924538-d0e80a76-d1ad-4dd0-ab4a-00aff1cb92ea" src="https://github.com/user-attachments/assets/9108cd53-59ce-4ee9-ad2d-2c1d395ec0b2" />

- Gemini Enterprise Agent Platform : https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/access-control?hl=zh-tw
<img width="958" height="910" alt="617926338-483ee9a0-c6d9-4f18-a7a6-515a5762adff" src="https://github.com/user-attachments/assets/74f2dd35-1eb3-4ae6-9905-d41e52ad1012" />



- AI.GENERATE_TEXT : https://docs.cloud.google.com/bigquery/docs/generate-text?hl=zh-tw
<img width="958" height="913" alt="617923436-eefb4d98-25aa-4a2b-bfbd-5d7d744af9ce" src="https://github.com/user-attachments/assets/f96729c2-e1fe-448c-80bf-efd6cd7b6377" />


- BigQuery AutoML Pricing : https://cloud.google.com/products/gemini-enterprise-agent-platform/pricing?hl=zh_tw
<img width="1920" height="911" alt="617912395-09686c77-7ee8-4674-8a84-21255376dcc3" src="https://github.com/user-attachments/assets/3563d30f-b967-4541-b779-124e8f106657" />


- Gemini model Pricing : 
https://cloud.google.com/gemini-enterprise-agent-platform/generative-ai/pricing?authuser=0&hl=zh-tw#standard_1
<img width="1920" height="911" alt="617911967-71930f0b-718b-4bd8-91b4-cf0f7d1b9432" src="https://github.com/user-attachments/assets/9f703a2e-c28f-4f7e-a4b4-4376fef63602" />


## Cost Comparison: Querying in BigQuery vs. Transferring to SQL Server

### 1. Cost Structure Comparison
| Cost Component | Querying in BigQuery | Transferring to SQL Server |
|----------------|----------------------|---------------------------|
| **Query/Compute Cost** | $6.25 per TB scanned for queries  | Pay once for the extraction (Storage Read API) |
| **Storage Cost** | ~$0.02/GB/month (active), ~$0.01/GB/month (long-term) | Your own SQL Server storage costs (disks, licensing) |
| **Data Transfer (Egress)** | **Separate cost** to export results to external systems | **Main cost factor** - transferring data out of Google Cloud  |
| **BigQuery Storage Read API** | Not applicable | **$1.10 per TB** read (after 300 TB free tier)  |
| **Batch Export Transfer** | Not applicable | **$0.08 per GB** for data transfer (after 300 TB/month free)  |

### 2. Key Insight: The Free Tier

BigQuery provides a **300 TB per month** free tier for data read via the Storage Read API . This means:

- **If you're under 300 TB/month**: Your data extraction to SQL Server could be **free** (only Storage Read API costs apply after that) 
- **Query costs ($6.25/TB)**: You always pay this for analysis queries, regardless of transfer 

> **Important**: Query costs **do not include** external data transfer charges. Egress fees apply separately when transferring data out of Google Cloud to external destinations like your SQL Server .

## Cost Breakdown: Transferring Data to SQL Server

### Storage Costs
| Storage Type | Cost | Details |
|--------------|------|---------|
| Active logical storage | ~$0.02/GB/month | Data modified in last 90 days  |
| Long-term logical storage | ~$0.01/GB/month | Data unchanged for 90+ days (automatic discount)  |
| Free Tier | First 10 GiB per month free | Both active and long-term storage  |

When transferring data from BigQuery to SQL Server, you're typically paying for:

###  BigQuery Data Extraction Cost

| Component | Cost |
|-----------|------|
| **Storage Read API** | $1.10 per TB (after 300 TB free/month)  |
| **Storage Read Data Transfer** | $0.08 per GB (after 300 TB free/month)  |

> **Example**: Transferring 100 GB per night:
> - Storage Read API: $1.10 × 0.1 = **$0.11**
> - Data Transfer: $0.08 × 100 = **$8.00**
> - **Total per night**: ~$8.11
> - **Monthly (30 days)**: ~$243


### Summary: Cost Comparison

| Scenario | Cost Model | Best For |
|----------|------------|----------|
| **Query in BigQuery** | $6.25/TB per query + storage | Ad-hoc, low-frequency analysis |
| **Transfer to SQL Server** | $0.08/GB + $1.10/TB (one-time per export) + SQL Server licensing/ops | Frequent, production queries |

## Google Cloud Official Documentation

- BigQuery Normal Pricing :  https://cloud.google.com/bigquery/pricing?hl=zh_tw#storage
<img width="958" height="910" alt="617936404-a0c6ac2a-4f03-42a5-92e1-f566d71a75c9" src="https://github.com/user-attachments/assets/409379c5-d75b-4765-8d2e-77b39ee403b9" />
<img width="958" height="910" alt="617936825-5fde1b30-5fa3-4cf6-b437-21e867842fa3" src="https://github.com/user-attachments/assets/7b77161d-f14c-4c3b-95c5-0659e1fe8d08" />

- BigQuery Storage Read API Pricing : https://cloud.google.com/bigquery/pricing?hl=zh_tw#storage-read-api
<img width="958" height="910" alt="617933912-55dc2889-8943-40ef-87cd-8b284077565c" src="https://github.com/user-attachments/assets/5e6f5a9b-f9c2-46cd-8e3b-d2601a0fe747" />
<img width="958" height="910" alt="617933142-84356236-6c15-44de-b8b1-f40c6e491bea" src="https://github.com/user-attachments/assets/0d8e0e1e-f1a5-404b-9170-dedc867ea7ea" />


- Batch Loading : https://docs.cloud.google.com/bigquery/docs/batch-loading-data?hl=zh-tw
<img width="958" height="910" alt="617934795-010e710a-c157-45f6-8e4b-a6837bc92411" src="https://github.com/user-attachments/assets/fa9c251d-1ed2-4d01-9512-e495084d7e9e" />
<img width="958" height="910" alt="617935751-415909c1-5c1a-4e26-8109-805c144e64a6" src="https://github.com/user-attachments/assets/909c2f5a-bdf5-4d08-9311-3b8c5e80030a" />



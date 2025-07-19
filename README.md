# WooCommerce Out of Stock Last

This code snippet provides a solution to **automatically move out-of-stock WooCommerce products to the end of product listings** on shop, category, and archive pages. By default, WooCommerce might display out-of-stock items mixed with available ones, which can be frustrating for customers. This modification ensures that available products are always shown first, improving the browsing experience.

## Why move out-of-stock products to the end?

* **Improved User Experience:** Customers see available products first, reducing frustration and making it easier to find purchasable items.
* **Enhanced Browsing:** Keeps the most relevant products (those in stock) at the top, leading to a more efficient shopping journey.
* **Better Conversion Potential:** By prioritizing in-stock items, you subtly guide users towards products they can immediately buy.
* **Cleaner Product Displays:** Prevents out-of-stock products from cluttering the top of your category and shop pages.

## Features

* Modifies the default WooCommerce product query to sort by stock status.
* Ensures products with `_stock_status` of 'instock' appear before 'outofstock' products.
* Maintains secondary sorting (e.g., by date or menu order) for products within the same stock status.
* Applies to WooCommerce shop, category, and archive pages.

## Installation

There are two primary methods to implement this code:

### Method 1: As a Standalone Plugin (Recommended)

Creating a small, dedicated plugin is the most robust way to add this functionality. It ensures your modification remains active regardless of theme changes.

1.  Create a new folder named `wc-out-of-stock-last` inside your WordPress site's `wp-content/plugins/` directory.
2.  Inside this new folder, create a file named `wc-out-of-stock-last.php`.
3.  Copy and paste the following code into `wc-out-of-stock-last.php`:

    ```php
    <?php
    /**
     * Plugin Name: WooCommerce Out of Stock Last
     * Description: Moves out-of-stock WooCommerce products to the end of product listings on shop and archive pages.
     * Version: 1.0
     * Author: Your Name (Optional)
     * License: GPL-2.0-or-later
     */

    if ( ! defined( 'ABSPATH' ) ) {
        exit; // Exit if accessed directly.
    }

    /**
     * Modifies the default catalog ordering arguments to prioritize in-stock products.
     *
     * @param array $args The query arguments.
     * @return array Modified query arguments.
     */
    add_filter('woocommerce_get_catalog_ordering_args', 'move_out_of_stock_products_to_end');

    function move_out_of_stock_products_to_end($args) {
        // Only apply if not already sorting by stock status or if a specific orderby is not set
        // This ensures compatibility with other sorting options
        if ( ! isset( $args['orderby'] ) || ( isset( $args['orderby'] ) && $args['orderby'] !== 'meta_value' ) ) {
            $args['orderby'] = array(
                'meta_value' => 'ASC', // 'instock' comes before 'outofstock' alphabetically
                'date'       => 'DESC', // Then sort by date, newest first
                'menu_order' => 'ASC',  // Finally by menu order
            );
            $args['meta_key'] = '_stock_status'; // Check stock status meta key
        }
        return $args;
    }

    /**
     * Further modifies the posts clauses to ensure out-of-stock products are moved to the end.
     * This filter is more powerful for direct SQL manipulation.
     *
     * @param array    $clauses The clauses of the query.
     * @param WP_Query $query   The WP_Query instance.
     * @return array Modified clauses.
     */
    add_filter('posts_clauses', 'modify_woocommerce_product_query_for_stock_status', 10, 2);

    function modify_woocommerce_product_query_for_stock_status($clauses, $query) {
        global $wpdb;

        // Ensure it's not in the admin, it's the main query, and it's a WooCommerce product query
        if ( is_admin() || ! $query->is_main_query() || ! is_post_type_archive( 'product' ) && ! $query->is_tax( get_object_taxonomies( 'product' ) ) ) {
            return $clauses;
        }

        // Add JOIN for _stock_status meta key
        $clauses['join'] .= " LEFT JOIN {$wpdb->postmeta} AS wc_stock_status ON ({$wpdb->posts}.ID = wc_stock_status.post_id AND wc_stock_status.meta_key = '_stock_status')";

        // Modify ORDER BY to prioritize 'instock'
        // This will place 'instock' items first (alphabetically 'i' before 'o')
        // and then apply the default WooCommerce sorting or other specified sorting.
        // We ensure that existing orderby is respected for items within the same stock status.
        $default_orderby = $clauses['orderby']; // Capture existing orderby if any
        $clauses['orderby'] = "wc_stock_status.meta_value ASC, " . $default_orderby;

        return $clauses;
    }
    ```

4.  Go to your WordPress admin dashboard, navigate to **"Plugins"**, and **activate** the "WooCommerce Out of Stock Last" plugin.

### Method 2: Adding to your Theme's functions.php File

You can add this code directly to your active theme's `functions.php` file. **Before doing so, it's highly recommended to back up your `functions.php` file.**

1.  Navigate to `wp-content/themes/YourThemeName/` (replace `YourThemeName` with the actual name of your active theme).
2.  Open the `functions.php` file.
3.  Add the following code to the end of the file (before the closing `?>` tag, if one exists):

    ```php
    /**
     * Modifies the default catalog ordering arguments to prioritize in-stock products.
     *
     * @param array $args The query arguments.
     * @return array Modified query arguments.
     */
    add_filter('woocommerce_get_catalog_ordering_args', 'move_out_of_stock_products_to_end');

    function move_out_of_stock_products_to_end($args) {
        // Only apply if not already sorting by stock status or if a specific orderby is not set
        // This ensures compatibility with other sorting options
        if ( ! isset( $args['orderby'] ) || ( isset( $args['orderby'] ) && $args['orderby'] !== 'meta_value' ) ) {
            $args['orderby'] = array(
                'meta_value' => 'ASC', // 'instock' comes before 'outofstock' alphabetically
                'date'       => 'DESC', // Then sort by date, newest first
                'menu_order' => 'ASC',  // Finally by menu order
            );
            $args['meta_key'] = '_stock_status'; // Check stock status meta key
        }
        return $args;
    }

    /**
     * Further modifies the posts clauses to ensure out-of-stock products are moved to the end.
     * This filter is more powerful for direct SQL manipulation.
     *
     * @param array    $clauses The clauses of the query.
     * @param WP_Query $query   The WP_Query instance.
     * @return array Modified clauses.
     */
    add_filter('posts_clauses', 'modify_woocommerce_product_query_for_stock_status', 10, 2);

    function modify_woocommerce_product_query_for_stock_status($clauses, $query) {
        global $wpdb;

        // Ensure it's not in the admin, it's the main query, and it's a WooCommerce product query
        if ( is_admin() || ! $query->is_main_query() || ! is_post_type_archive( 'product' ) && ! $query->is_tax( get_object_taxonomies( 'product' ) ) ) {
            return $clauses;
        }

        // Add JOIN for _stock_status meta key
        $clauses['join'] .= " LEFT JOIN {$wpdb->postmeta} AS wc_stock_status ON ({$wpdb->posts}.ID = wc_stock_status.post_id AND wc_stock_status.meta_key = '_stock_status')";

        // Modify ORDER BY to prioritize 'instock'
        // This will place 'instock' items first (alphabetically 'i' before 'o')
        // and then apply the default WooCommerce sorting or other specified sorting.
        // We ensure that existing orderby is respected for items within the same stock status.
        $default_orderby = $clauses['orderby']; // Capture existing orderby if any
        $clauses['orderby'] = "wc_stock_status.meta_value ASC, " . $default_orderby;

        return $clauses;
    }
    ```

## Important Considerations

* **WooCommerce Setting:** WooCommerce has a built-in setting under **WooCommerce > Settings > Products > Inventory > Out of Stock visibility** to "Hide out of stock items from the catalog." If this option is checked, out-of-stock products will be completely hidden, rendering this code unnecessary. This snippet is for when you *do* want to show out-of-stock products but prefer them at the end.
* **Performance:** For very large stores with tens of thousands of products, modifying queries can sometimes have a minor impact on performance. However, for most stores, this snippet should work efficiently.
* **Compatibility:** This code uses standard WordPress and WooCommerce filters. While generally compatible, always test thoroughly with your specific theme and other plugins.
* **Sorting Order:** The `meta_value` for `_stock_status` is 'instock' or 'outofstock'. Sorting `ASC` (ascending) will naturally place 'instock' before 'outofstock' because 'i' comes before 'o' alphabetically.

## Contributing

Contributions are welcome! If you have suggestions or improvements for this code, feel free to open a "Pull Request" or report an "Issue."

## License

This project is licensed under the GPL-2.0-or-later License.

# محصولات ناموجود ووکامرس در انتها

این قطعه کد راه حلی برای **انتقال خودکار محصولات ناموجود (Out-of-Stock) ووکامرس به انتهای لیست محصولات** در صفحات فروشگاه، دسته‌بندی و آرشیو فراهم می‌کند. به طور پیش‌فرض، ووکامرس ممکن است اقلام ناموجود را با اقلام موجود مخلوط نمایش دهد که می‌تواند برای مشتریان ناامیدکننده باشد. این تغییر تضمین می‌کند که محصولات موجود همیشه ابتدا نمایش داده می‌شوند و تجربه مرور را بهبود می‌بخشد.

## چرا محصولات ناموجود را به انتها منتقل کنیم؟

* **بهبود تجربه کاربری:** مشتریان ابتدا محصولات موجود را مشاهده می‌کنند، که از ناامیدی جلوگیری کرده و یافتن اقلام قابل خرید را آسان‌تر می‌کند.
* **مرور پیشرفته:** مرتبط‌ترین محصولات (آن‌هایی که در انبار موجود هستند) را در بالا نگه می‌دارد و به یک سفر خرید کارآمدتر منجر می‌شود.
* **پتانسیل تبدیل بهتر:** با اولویت‌بندی اقلام موجود در انبار، کاربران را به طور ظریفی به سمت محصولاتی که می‌توانند بلافاصله خریداری کنند، هدایت می‌کنید.
* **نمایش محصول تمیزتر:** از شلوغ شدن بالای صفحات دسته‌بندی و فروشگاه شما توسط محصولات ناموجود جلوگیری می‌کند.

## قابلیت‌ها

* کوئری پیش‌فرض محصولات ووکامرس را برای مرتب‌سازی بر اساس وضعیت موجودی تغییر می‌دهد.
* اطمینان حاصل می‌کند که محصولات با `_stock_status` 'instock' قبل از محصولات 'outofstock' ظاهر می‌شوند.
* مرتب‌سازی ثانویه (مانند بر اساس تاریخ یا ترتیب منو) را برای محصولات با وضعیت موجودی یکسان حفظ می‌کند.
* در صفحات فروشگاه، دسته‌بندی و آرشیو ووکامرس اعمال می‌شود.

## نصب

برای پیاده‌سازی این کد، دو روش اصلی وجود دارد:

### روش ۱: به عنوان یک افزونه مستقل (توصیه شده)

ایجاد یک افزونه کوچک و اختصاصی، قوی‌ترین راه برای افزودن این قابلیت است. این تضمین می‌کند که تغییر شما صرف‌نظر از تغییر قالب، فعال باقی بماند.

1.  یک پوشه جدید با نام `wc-out-of-stock-last` در مسیر `wp-content/plugins/` سایت وردپرسی خود ایجاد کنید.
2.  در داخل این پوشه جدید، یک فایل با نام `wc-out-of-stock-last.php` ایجاد کنید.
3.  کد زیر را در `wc-out-of-stock-last.php` کپی و جایگذاری کنید:

    ```php
    <?php
    /**
     * Plugin Name: WooCommerce Out of Stock Last
     * Description: Moves out-of-stock WooCommerce products to the end of product listings on shop and archive pages.
     * Version: 1.0
     * Author: Your Name (Optional)
     * License: GPL-2.0-or-later
     */

    if ( ! defined( 'ABSPATH' ) ) {
        exit; // Exit if accessed directly.
    }

    /**
     * Modifies the default catalog ordering arguments to prioritize in-stock products.
     *
     * @param array $args The query arguments.
     * @return array Modified query arguments.
     */
    add_filter('woocommerce_get_catalog_ordering_args', 'move_out_of_stock_products_to_end');

    function move_out_of_stock_products_to_end($args) {
        // فقط در صورتی اعمال شود که مرتب‌سازی بر اساس وضعیت موجودی قبلاً تنظیم نشده باشد یا یک orderby خاص تنظیم نشده باشد
        // این کار سازگاری با سایر گزینه‌های مرتب‌سازی را تضمین می‌کند
        if ( ! isset( $args['orderby'] ) || ( isset( $args['orderby'] ) && $args['orderby'] !== 'meta_value' ) ) {
            $args['orderby'] = array(
                'meta_value' => 'ASC', // 'instock' قبل از 'outofstock' به ترتیب حروف الفبا می‌آید
                'date'       => 'DESC', // سپس بر اساس تاریخ، جدیدترین اول
                'menu_order' => 'ASC',  // در نهایت بر اساس ترتیب منو
            );
            $args['meta_key'] = '_stock_status'; // بررسی کلید متا وضعیت موجودی
        }
        return $args;
    }

    /**
     * Further modifies the posts clauses to ensure out-of-stock products are moved to the end.
     * This filter is more powerful for direct SQL manipulation.
     *
     * @param array    $clauses The clauses of the query.
     * @param WP_Query $query   The WP_Query instance.
     * @return array Modified clauses.
     */
    add_filter('posts_clauses', 'modify_woocommerce_product_query_for_stock_status', 10, 2);

    function modify_woocommerce_product_query_for_stock_status($clauses, $query) {
        global $wpdb;

        // اطمینان از اینکه در بخش مدیریت نیست، یک کوئری اصلی است، و یک کوئری محصول ووکامرس است
        if ( is_admin() || ! $query->is_main_query() || ! is_post_type_archive( 'product' ) && ! $query->is_tax( get_object_taxonomies( 'product' ) ) ) {
            return $clauses;
        }

        // اضافه کردن JOIN برای کلید متا _stock_status
        $clauses['join'] .= " LEFT JOIN {$wpdb->postmeta} AS wc_stock_status ON ({$wpdb->posts}.ID = wc_stock_status.post_id AND wc_stock_status.meta_key = '_stock_status')";

        // تغییر ORDER BY برای اولویت‌بندی 'instock'
        // این کار اقلام 'instock' را ابتدا قرار می‌دهد (به ترتیب حروف الفبا 'i' قبل از 'o' می‌آید)
        // و سپس مرتب‌سازی پیش‌فرض ووکامرس یا سایر مرتب‌سازی‌های مشخص شده را اعمال می‌کند.
        // ما اطمینان می‌دهیم که orderby موجود برای اقلام با وضعیت موجودی یکسان رعایت شود.
        $default_orderby = $clauses['orderby']; // دریافت orderby موجود در صورت وجود
        $clauses['orderby'] = "wc_stock_status.meta_value ASC, " . $default_orderby;

        return $clauses;
    }
    ```

4.  وارد پنل مدیریت وردپرس خود شوید، به بخش **"افزونه‌ها"** بروید و افزونه **"محصولات ناموجود ووکامرس در انتها"** را **فعال کنید**.

### روش ۲: اضافه کردن به فایل functions.php قالب شما

می‌توانید این کد را مستقیماً به فایل `functions.php` قالب فعال خود اضافه کنید. **پیشنهاد اکید می‌شود قبل از انجام این کار، از فایل `functions.php` خود یک پشتیبان (backup) تهیه کنید.**

1.  به مسیر `wp-content/themes/YourThemeName/` بروید (به جای `YourThemeName` نام واقعی قالب فعال خود را قرار دهید).
2.  فایل `functions.php` را باز کنید.
3.  کد زیر را به انتهای فایل (قبل از تگ بستن `?>`، در صورت وجود) اضافه کنید:

    ```php
    /**
     * Modifies the default catalog ordering arguments to prioritize in-stock products.
     *
     * @param array $args The query arguments.
     * @return array Modified query arguments.
     */
    add_filter('woocommerce_get_catalog_ordering_args', 'move_out_of_stock_products_to_end');

    function move_out_of_stock_products_to_end($args) {
        // فقط در صورتی اعمال شود که مرتب‌سازی بر اساس وضعیت موجودی قبلاً تنظیم نشده باشد یا یک orderby خاص تنظیم نشده باشد
        // این کار سازگاری با سایر گزینه‌های مرتب‌سازی را تضمین می‌کند
        if ( ! isset( $args['orderby'] ) || ( isset( $args['orderby'] ) && $args['orderby'] !== 'meta_value' ) ) {
            $args['orderby'] = array(
                'meta_value' => 'ASC', // 'instock' قبل از 'outofstock' به ترتیب حروف الفبا می‌آید
                'date'       => 'DESC', // سپس بر اساس تاریخ، جدیدترین اول
                'menu_order' => 'ASC',  // در نهایت بر اساس ترتیب منو
            );
            $args['meta_key'] = '_stock_status'; // بررسی کلید متا وضعیت موجودی
        }
        return $args;
    }

    /**
     * Further modifies the posts clauses to ensure out-of-stock products are moved to the end.
     * This filter is more powerful for direct SQL manipulation.
     *
     * @param array    $clauses The clauses of the query.
     * @param WP_Query $query   The WP_Query instance.
     * @return array Modified clauses.
     */
    add_filter('posts_clauses', 'modify_woocommerce_product_query_for_stock_status', 10, 2);

    function modify_woocommerce_product_query_for_stock_status($clauses, $query) {
        global $wpdb;

        // اطمینان از اینکه در بخش مدیریت نیست، یک کوئری اصلی است، و یک کوئری محصول ووکامرس است
        if ( is_admin() || ! $query->is_main_query() || ! is_post_type_archive( 'product' ) && ! $query->is_tax( get_object_taxonomies( 'product' ) ) ) {
            return $clauses;
        }

        // اضافه کردن JOIN برای کلید متا _stock_status
        $clauses['join'] .= " LEFT JOIN {$wpdb->postmeta} AS wc_stock_status ON ({$wpdb->posts}.ID = wc_stock_status.post_id AND wc_stock_status.meta_key = '_stock_status')";

        // تغییر ORDER BY برای اولویت‌بندی 'instock'
        // این کار اقلام 'instock' را ابتدا قرار می‌دهد (به ترتیب حروف الفبا 'i' قبل از 'o' می‌آید)
        // و سپس مرتب‌سازی پیش‌فرض ووکامرس یا سایر مرتب‌سازی‌های مشخص شده را اعمال می‌کند.
        // ما اطمینان می‌دهیم که orderby موجود برای اقلام با وضعیت موجودی یکسان رعایت شود.
        $default_orderby = $clauses['orderby']; // دریافت orderby موجود در صورت وجود
        $clauses['orderby'] = "wc_stock_status.meta_value ASC, " . $default_orderby;

        return $clauses;
    }
    ```

## ملاحظات مهم

* **تنظیمات ووکامرس:** ووکامرس یک تنظیم داخلی در مسیر **ووکامرس > تنظیمات > محصولات > موجودی > قابلیت مشاهده محصولات ناموجود در انبار** دارد که می‌توانید "پنهان کردن اقلام ناموجود از کاتالوگ" را فعال کنید. اگر این گزینه فعال باشد، محصولات ناموجود به طور کامل پنهان می‌شوند و این کد غیرضروری خواهد بود. این قطعه کد برای زمانی است که شما *می‌خواهید* محصولات ناموجود را نمایش دهید اما ترجیح می‌دهید آن‌ها در انتها قرار گیرند.
* **عملکرد:** برای فروشگاه‌های بسیار بزرگ با ده‌ها هزار محصول، تغییر کوئری‌ها ممکن است گاهی اوقات تأثیر جزئی بر عملکرد داشته باشد. با این حال، برای اکثر فروشگاه‌ها، این قطعه کد باید به طور کارآمد عمل کند.
* **سازگاری:** این کد از فیلترهای استاندارد وردپرس و ووکامرس استفاده می‌کند. در حالی که به طور کلی سازگار است، همیشه با قالب خاص و سایر افزونه‌های خود به طور کامل آزمایش کنید.
* **ترتیب مرتب‌سازی:** `meta_value` برای `_stock_status` 'instock' یا 'outofstock' است. مرتب‌سازی `ASC` (صعودی) به طور طبیعی 'instock' را قبل از 'outofstock' قرار می‌دهد زیرا 'i' قبل از 'o' به ترتیب حروف الفبا می‌آید.

## مشارکت (Contributing)

مشارکت شما خوشایند است! اگر پیشنهاد یا بهبودهایی برای این کد دارید، می‌توانید یک "Pull Request" ایجاد کنید یا "Issue" جدیدی را گزارش دهید.

## مجوز (License)

این پروژه تحت مجوز GPL-2.0-or-later منتشر شده است.

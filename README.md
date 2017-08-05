# Contact7ticket
# Add 100 ticket checkbox on contact form.
# create database table on activation of this plugin and add defulat vaue in it.

```
function create_plugin_database_table()
{
   global $table_prefix, $wpdb;
   $tblname = 'contact_ticket';
    $wp_track_table = $table_prefix . "$tblname";
    if($wpdb->get_var( "show tables like '$wp_track_table'" ) != $wp_track_table) 
    {
        $sql = "CREATE TABLE `". $wp_track_table . "` ( ";
        for($i=1;$i<=100;$i++){
			$sql .= "`ticket" . $i ."` varchar(100)   NOT NULL";
			if($i != 100){
				$sql .= ", ";
			}
		}
		$sql .= ") ENGINE=MyISAM DEFAULT CHARSET=latin1; ";
        require_once( ABSPATH . '/wp-admin/includes/upgrade.php' );
        dbDelta($sql);
    }
}
function ticket_install_data() {
	global $wpdb;
	$array = array();
	$array['user_id'] = $user_id;
	$table_name = $wpdb->prefix . 'contact_ticket';
	for($i=1;$i<=100;$i++){
			$array["ticket$i"] = 0;
	}
	$wpdb->insert( 
		$table_name, 
		$array
	);
}
 register_activation_hook( __FILE__, 'create_plugin_database_table' );
 register_activation_hook( __FILE__, 'ticket_install_data' );
```


# add shortcode for contact form for admin side

```
if ( !function_exists( 'wpcf7_add_form_tag' ) ) { 
    require_once( ABSPATH . 'wp-content/plugins/contact-form-7/includes/form-tags-manager.php'); 
} 
  ```  
  
  
# On front side of contact form we add checkbox if shortcode is add in contact form area.


 ```  
function wpcf7_custom_ticket_book() {
	global $wpdb;
	$tablename = $wpdb->prefix . 'contact_ticket';
	wp_enqueue_style('ContactAddon', plugins_url( 'css/contact_addon.css' , __FILE__ ), false, '1.0', 'all' );
    $html='';
	if (is_user_logged_in()) {
		$user = wp_get_current_user();
		$results = $wpdb->get_results("SELECT * FROM $tablename WHERE user_id = '".(int) $user->ID."'",ARRAY_A);
		if($wpdb->num_rows > 0) {
			$result = $results[0];
			for($i=1;$i<=100;$i++){
				$disabled='';
				if($result["ticket$i"] == 1)
				{
					$disabled = 'disabled';
				}
				$html .= '<span class="checkbox-wrpaper"><input type="checkbox" '. $disabled .' name="ticket' . $i . '" value="' . 1 . '" /> Ticket' . $i. "</span>";
			}
		}else{
			for($i=1;$i<=100;$i++){
			$html .= '<span class="checkbox-wrpaper"><input type="checkbox" name="ticket' . $i . '" value="' . 1 . '" /> Ticket' . $i. "</span>";
			}	
		}
	}else{
		for($i=1;$i<=100;$i++){
			$html .= '<span class="checkbox-wrpaper"><input type="checkbox" name="ticket' . $i . '" value="' . 1 . '" /> Ticket' . $i. "</span>";
			}	
		}
	 return $html;
	
}	

wpcf7_add_form_tag('ticket_book_cf7', 'wpcf7_custom_ticket_book', true);
```     

# save booked ticket to database on form submit For that we use hook wpcf7_before_send_mail
 
 ```  
function save_form( $wpcf7 ) {
	global $wpdb;
	global $current_user;
	$table_name = $wpdb->prefix . 'contact_ticket';
   $submission = WPCF7_Submission::get_instance();
  if ( $submission ) {
		$submited = array();
		$submited['posted_data'] = $submission->get_posted_data();
		$array = array();
		if (user_id != 0 ) {
			$array['user_id'] = user_id;
			$results = $wpdb->get_results("SELECT * FROM $table_name WHERE user_id = '". user_id."'",ARRAY_A);
			if($wpdb->num_rows > 0) {
				$result = $results[0];
				for($i=1;$i<=100;$i++){
					if(isset($submited['posted_data']["ticket$i"])){
						$array["ticket$i"] = $submited['posted_data']["ticket$i"];
					}
					elseif($result["ticket$i"] == 1){
						$array["ticket$i"] = 1;
					}
					else{
						$array["ticket$i"] = 0;
					}
				}
				$wpdb->delete( $table_name, array( 'user_id' => user_id ) );
			}else{
				for($i=1;$i<=100;$i++){
					if(isset($submited['posted_data']["ticket$i"])){
						$array["ticket$i"] = $submited['posted_data']["ticket$i"];
					}else{
						$array["ticket$i"] = 0 ;
					}
				}
			}
			
		}
		else{
			$array['user_id'] = 0;
			for($i=1;$i<=100;$i++){
				if(isset($submited['posted_data']["ticket$i"])){
					$array["ticket$i"] = $submited['posted_data']["ticket$i"];
				}else{
					$array["ticket$i"] = 0;
				}
			}
		}
		
		$wpdb->insert( $table_name, $array);
   }		
}
add_action('wpcf7_before_send_mail', 'save_form' );
```  


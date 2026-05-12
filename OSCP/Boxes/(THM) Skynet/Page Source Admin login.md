
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Cuppa CMS</title>
<!-- Principals Packages  -->
    <link href="templates/default/css/tu_main.css" rel="stylesheet" type="text/css">
    <link href="templates/default/css/template.css" rel="stylesheet" type="text/css">
    <link href="templates/default/css/jquery-ui.css" rel="stylesheet" type="text/css">
    <link href="js/uploadify/uploadify.css" type="text/css" rel="stylesheet" />
    <script src="js/jquery.js" type="text/javascript"></script>
    <script src="js/jquery-ui.js" type="text/javascript"></script>
    <script src="js/jquery.validate.js" type="text/javascript"></script>
    <script src="js/jquery.md5.js" type="text/javascript"></script>
    <script src="js/jquery.sha1.js" type="text/javascript"></script>
    <script src="js/uploadify/jquery.uploadify.js" type="text/javascript"></script>
    <script src="js/tiny_mce/tiny_mce.js" type="text/javascript"></script>
    <script src="js/swfobject.js" type="text/javascript"></script>
    <script src="js/tu_main.js" type="text/javascript"></script>
<!-- Others Packages -->
    <link href="templates/default/css/alertConfigField.css" rel="stylesheet" type="text/css">
    <link href="templates/default/css/defaultAlert.css" rel="stylesheet" type="text/css">
    <link href="templates/default/css/alertIFrame.css" rel="stylesheet" type="text/css">
    <link href="templates/default/css/alertImage.css" rel="stylesheet" type="text/css">
    <link href="templates/default/css/dropdown.css" rel="stylesheet" type="text/css">
    <link href="templates/default/css/datagrid.css" rel="stylesheet" type="text/css">
    <link href="templates/default/css/form.css" rel="stylesheet" type="text/css">
    <link href="templates/default/css/tools.css" rel="stylesheet" type="text/css">
    
    <script src="js/tu_stringHelp.js" type="text/javascript"></script>
    <script src="js/tu_comboBoxHelp.js" type="text/javascript"></script>
    <script src="js/tu_checkBoxHelp.js" type="text/javascript"></script>
    <script src="js/tu_functions.js" type="text/javascript"></script>    </head>
    <body>
        <style type="text/css">
	body{
		height:100%;
		background:url(templates/default/images/template/login/login_bg.jpg);
		background-position: center 0px;
	}
</style>
<div class="login" id="login">
	<div class="logo"></div>
    <div class="login_box">
    	<form id="form_login" method="post" style="display:block">
            <div class="comment">Use a valid username and password to gain access to the administrator</div>
            <table class="table" style="width:400px;">
                <tr>
                    <td style="font-weight:bold">Username</td>
                    <td><input type="text" name="user" id="user" class="input" value="" /></td>
                </tr>
                <tr>
                    <td style="font-weight:bold">Password</td>
                    <td><input type="password" name="password" id="password" class="input" value="" /></td>
                </tr>
                <tr>
                    <td></td>
                    <td><input class="button_submit" type="submit" value="" /></td>
                </tr>
                <input type="hidden" id="task" name="task" value="login"/>
            </table>
            <!--
            	<a class="forgot_password" onclick="ShowPanel('forget')">Forgot Password?</a>
            -->
        </form>
		<form id="form_forget" method="post" style="display:none">
        	<div class="comment">Please send us your email and we'll reset your password</div>
            <table class="table" style="top:110px; left:90px">
                <tr>
                    <td style="font-weight:bold">Email address</td>
                    <td><input type="text" name="email" id="email" class="input" value="" /></td>
                </tr>
                <tr>
                    <td></td>
                    <td><input class="button_submit" type="submit" value="" /></td>
                </tr>
                <input type="hidden" id="task" name="task" value="forgot"/>
            </table>
            <a class="back_to_login" onclick="ShowPanel('login')"><img src="../administrator/templates/default/images/template/login/icon_back_login.png" style="margin-right:5px;" />Back to login</a>
        </form>
    </div>
</div>
<script>
	function ShowPanel(value){
		if(value == "login"){
			jQuery("#form_login").css("display", "block");
			jQuery("#form_forget").css("display", "none");
		}else if(value == "forget"){
			jQuery("#form_login").css('display', "none");
			jQuery("#form_forget").css("display", "block");
		}
	}
</script>    </body>
</html>
# PDF Generation via DOMPDF Library

http://code.google.com/p/dompdf/

Input:

 * HTML string (which could be rendered template)
 * HTML File
 
Output

 * PDF File location
 * SS File
 * PDF binary stream to browser

## Quickly getting started

Find the ID number of a page (ie 1) and go to; /pdf/1

Customize your tempalte by copying DefaultPDF to your theme directory.

View raw output by going to; /pdf/1?show


## Example usage

	$pdf = new SS_DOMPDF();
	$pdf->setHTML($mydataobject->renderWith('MyTemplate'));
	$pdf->render();
	$pdf->toFile('mypdf.pdf');
	
## Debugging

The $pdf->streamdebug(); function is useful for quickly viewing pdfs, particularly
if your browser supports displaying pdfs, rather than downloading.

You can check your html before it is converted like this:

	echo $mydataobject->renderWith('MyTemplate');die();

Once this has been installed then you need to;
	parse your PDF templates to a .html file
	get dompdf to convert the saved .html file
	
	
A more advanced solution would be something like;

class CustomPage_Controller extends Page_Controller {

	public function mypdf(){
		if($member = Member::currentUser()){
			Requirements::clear();
			
			if(!file_exists(ASSETS_PATH."/private")) mkdir(ASSETS_PATH."/private");
			
			require_once 'Zend/Date.php';
			$defaultDateFormat =Zend_Date::now()->toString($member->DateFormat);
			$defaultTimeFormat = Zend_Date::now()->toString($member->TimeFormat);			
			
			$content = $this->customise(array(
				'Member'	=> $member
			))->renderWith(array('pdf'));

			$filename=ASSETS_PATH."/private/current-pdf-{$member->ID}.html";
			$baseFile = preg_replace('/\\.pdf$/','',$filename);

			$fh = fopen($baseFile, "w+") or user_error("Couldn't open $baseFile.html for writing", E_USER_ERROR);
			fwrite($fh, $content) or user_error("Couldn't write content to $baseFile.html", E_USER_ERROR);
			fclose($fh);

			$dompdf = new SS_DOMPDF();
			$dompdf->load_html_file(ASSETS_PATH."/private/current-pdf-{$member->ID}.html");

			if ( isset($base_path) ) {$dompdf->set_base_path($base_path);}

			$paper = DOMPDF_DEFAULT_PAPER_SIZE;
			$orientation = "portrait";

			$dompdf->set_paper($paper, $orientation);
			$dompdf->render();
			
			
			$outfile = substr("{$member->FullName()}_{$defaultDateFormat}_{$defaultTimeFormat}", 0, 250).".pdf";
	
			$dompdf->stream(Convert::raw2xml($outfile));
		}	
	}
	
	/**
	* You may also like this 'test' function
	**/
	public function testpdf(){
		if($member = Member::currentUser()){
			Requirements::clear();
			require_once 'Zend/Date.php';
			$defaultDateFormat =Zend_Date::now()->toString($member->DateFormat);
			$defaultTimeFormat = Zend_Date::now()->toString($member->TimeFormat);			
			
			$property=$member->SavedProperty();
			$content = $this->customise(array(
				'Member'	=> $member
			))->renderWith(array('pdf'));
			return $content;
		}
	}
	
}

<?php

use IA_Designer\VectorGraphics\Geometry\Line;
use IA_Designer\VectorGraphics\Geometry\Point;
use IA_Designer\VectorGraphics\Graphics;
use IA_Designer\VectorGraphics\Style\Colour;
use IA_Designer\VectorGraphics\TagSVG\Text;
use Monolog\Handler\StreamHandler;
use Monolog\Logger;

include_once __DIR__ . "/../src/autoload.php";
include_once __DIR__ . "/../vendor/autoload.php";


$whoops = new \Whoops\Run;
$whoops->pushHandler(new \Whoops\Handler\PrettyPageHandler);
$whoops->register();

// Setup Monolog logger
$log = new Logger('Vector');
$log->pushHandler(new StreamHandler(__DIR__ . '/../../log/error2.log', Logger::WARNING));

function sum_prod($ar1, $ar2)
{
    $count = count($ar1);
    $sum = 0;
    if ($count !== count($ar2)) {
        return 0;
    }
    for ($i = 0; $i < $count; $i++) {
        $sum += $ar1[$i] * $ar2[$i];
    }
    return $sum;
};

$bins_ave = [];
$bins_decay = [];
$bins_NI = [];
$bins_max = [];

$dec_weights = [];

$num_grdes = isset($_POST["num"]) ? $_POST["num"] : 10;

$half_life = isset($_POST["half"]) ? $_POST["half"] : 5;

$num_sims = isset($_POST["sims"]) ? min($_POST["sims"], 30000 ): 3000;

$increase = isset($_POST["inc"]) ? $_POST["inc"] === "on" : false;

$above_50 = isset($_POST["fifty"]) ? $_POST["fifty"] === "on" : false;

$k = log(2) / $half_life;

for ($i = 0; $i < $num_grdes; $i++) {
    $dec_weights[] = exp(-$k * $i) * 1 / $num_grdes;
}

//$dec_weights = array_reverse($dec_weights);
$sum_dec = array_sum($dec_weights);

$sum_ave = 0;
$sum_decay = 0;
$sum_ni = 0;
$sum_max = 0;


for ($i = 0; $i < $num_sims; $i++) {
    $scores = [];
    for ($j = 0; $j < $num_grdes; $j++) {
        $scores[] = ($above_50) ? 50 + 50 * mt_rand() / mt_getrandmax()
            : 100 * mt_rand() / mt_getrandmax();
    }

    if ($increase) {
        rsort($scores);
    }

    //Standard Average model

    $grade = array_sum($scores) / $num_grdes;
    $grade_ave = $grade;


    $color = Colour::convertToSVGColor(Colour::hsvToRgb(
        (($grade * 360 / 100) + 235) % 360, 100, 100));


    $bins_ave[floor($grade)][] = [$grade_ave, $color, $grade];

    $sum_ave += $grade;


    //Decay model
    $grade = sum_prod($scores, $dec_weights) / $sum_dec;

    $bins_decay[floor($grade)][] = [$grade_ave, $color, $grade];

    $sum_decay += $grade;

    //Max Model

    $grade = max($scores);

    $bins_max[floor($grade)][] = [$grade_ave, $color, $grade];
    $sum_max += $grade;

    //Non-decreasing model
    //last, because we change the scores
    $max = 0;
    for ($j = 0; $j < $num_grdes; $j++) {
        $max = max($max, $scores[$j]);
        $scores[$j] = $max;
    }

    $grade = array_sum($scores) / $num_grdes;

    $bins_NI[floor($grade)][] = [$grade_ave, $color, $grade];
    $sum_ni += $grade;

}

$_comparator = function ($a, $b) {
    if ($a[0] === $b[0]) {
        return 0;
    }
    return ($a[0] < $b[0]) ? -1 : 1;
};

$sum_ave_mean = $sum_ave / $num_sims;
$sum_decay_mean = $sum_decay / $num_sims;
$sum_ni_mean = $sum_ni / $num_sims;
$sum_max_mean = $sum_max / $num_sims;

$graphics = new Graphics();

ksort($bins_ave);
foreach ($bins_ave as $gr => $grade) {
//    usort($grade, $_comparator);
    for ($i = 0, $l = count($grade); $i < $l; $i++) {
        $elements[] = ((new point(2 * $gr, 500 - $i * 2))->setColor($grade[$i][1]));
    }
}

$elements[] = Line::newLine([0, 501], [200, 501]);
$elements[] = Line::newLine([0, 501], [0, 501 + 5]);
$elements[] = Line::newLine([100, 501], [100, 501 + 5]);
$elements[] = Line::newLine([200, 501], [200, 501 + 5]);
$elements[] = Line::newLine([2 * $sum_ave_mean, 505], [2 * $sum_ave_mean, 0]);

$text_name = (new Text($graphics))->appendText("Average Model")->setAttribute("text-anchor", "middle");
$text_name->setLocation(new Point(100, 501+15));

$text_ave = (new Text($graphics))->appendText("Mean: ".round($sum_ave_mean, 2))->setAttribute("text-anchor", "middle");
$text_ave->setLocation(new Point(100, 501+30));
$elements[] = $text_name;
$elements[] = $text_ave;

ksort($bins_decay);
foreach ($bins_decay as $gr => $grade) {
    usort($grade, $_comparator);
    for ($i = 0, $l = count($grade); $i < $l; $i++) {
        $elements[] = ((new point(2 * $gr + 250, 500 - $i * 2))->setColor($grade[$i][1]));
    }

}
$elements[] = Line::newLine([250, 501], [200 + 250, 501]);
$elements[] = Line::newLine([250, 501], [250, 501 + 5]);
$elements[] = Line::newLine([350, 501], [350, 501 + 5]);
$elements[] = Line::newLine([450, 501], [450, 501 + 5]);

$elements[] = Line::newLine([250 + 2 * $sum_decay_mean, 505], [250 + 2 * $sum_decay_mean, 0]);
$text_name = (new Text($graphics))->appendText("Decay Model")->setAttribute("text-anchor", "middle");
$text_name->setLocation(new Point(350, 501+15));

$text_ave = (new Text($graphics))->appendText("Mean: ".round($sum_decay_mean, 2))->setAttribute("text-anchor", "middle");
$text_ave->setLocation(new Point(350, 501+30));
$text_bias = (new Text($graphics))->appendText("Bias: ".round($sum_decay_mean - $sum_ave_mean, 2))->setAttribute("text-anchor", "middle");
$text_bias->setLocation(new Point(350, 501+45));

$elements[] = $text_name;
$elements[] = $text_ave;
$elements[] = $text_bias;



ksort($bins_NI);
foreach ($bins_NI as $gr => $grade) {
    usort($grade, $_comparator);
    for ($i = 0, $l = count($grade); $i < $l; $i++) {
        $elements[] = ((new point(2 * $gr + 500, 500 - $i * 2))->setColor($grade[$i][1]));
    }

}

$elements[] = Line::newLine([500, 501], [200 + 500, 501]);
$elements[] = Line::newLine([500, 501], [500, 501 + 5]);
$elements[] = Line::newLine([600, 501], [600, 501 + 5]);
$elements[] = Line::newLine([700, 501], [700, 501 + 5]);
$elements[] = Line::newLine([500 + 2 * $sum_ni_mean, 505], [500 + 2 * $sum_ni_mean, 0]);


$elements[] = Line::newLine([250 + 2 * $sum_decay_mean, 505], [250 + 2 * $sum_decay_mean, 0]);
$text_name = (new Text($graphics))->appendText("Non-increasing (NI) Model")->setAttribute("text-anchor", "middle");
$text_name->setLocation(new Point(600, 501+15));

$text_ave = (new Text($graphics))->appendText("Mean: ".round($sum_ni_mean, 2))->setAttribute("text-anchor", "middle");
$text_ave->setLocation(new Point(600, 501+30));

$text_bias = (new Text($graphics))->appendText("Bias: ".round($sum_ni_mean - $sum_ave_mean, 2))->setAttribute("text-anchor", "middle");
$text_bias->setLocation(new Point(600, 501+45));


$elements[] = $text_name;
$elements[] = $text_ave;
$elements[] = $text_bias;

ksort($bins_max);
foreach ($bins_max as $gr => $grade) {
    usort($grade, $_comparator);
    for ($i = 0, $l = count($grade); $i < $l; $i++) {
        $elements[] = ((new point(2 * $gr + 750, 500 - $i * 2))->setColor($grade[$i][1]));
    }

}

$elements[] = Line::newLine([750, 501], [200 + 750, 501]);
$elements[] = Line::newLine([750, 501], [750, 501 + 5]);
$elements[] = Line::newLine([850, 501], [850, 501 + 5]);
$elements[] = Line::newLine([950, 501], [950, 501 + 5]);
$elements[] = Line::newLine([750 + 2 * $sum_ni_mean, 505], [750 + 2 * $sum_ni_mean, 0]);

$elements[] = Line::newLine([250 + 2 * $sum_decay_mean, 505], [250 + 2 * $sum_decay_mean, 0]);
$text_name = (new Text($graphics))->appendText("MAX Model")->setAttribute("text-anchor", "middle");
$text_name->setLocation(new Point(850, 501+15));

$text_ave = (new Text($graphics))->appendText("Mean: ".round($sum_max_mean, 2))->setAttribute("text-anchor", "middle");
$text_ave->setLocation(new Point(850, 501+30));

$text_bias = (new Text($graphics))->appendText("Bias: ".round($sum_max_mean - $sum_ave_mean, 2))->setAttribute("text-anchor", "middle");
$text_bias->setLocation(new Point(850, 501+45));

$elements[] = $text_name;
$elements[] = $text_ave;
$elements[] = $text_bias;


$graphics->setSvg($elements);

?>
    <style>
        input[type=number]{
            width: 60px;
        }

        div#Samples{
            position: absolute;
            left: 10px;
        }

        div#Num_ass{
            position: absolute;
            left: 145px;
        }

        div#Half_life{
            position: absolute;
            left: 370px;
        }

        div#Improve{
            position: absolute;
            left: 10px;
            top: 50px;
        }
        div#Above{
            position: absolute;
            left: 200px;
            top: 50px;
        }
        div.button{
            position: absolute;
            left: 1px;
            top: 80px;
        }

    </style>

    <form method="post">
        <div id = "Samples">
            <label for="half">Samples:</label>
            <input type="number" id="sims" name="sims" value="<?php echo $num_sims ?>">
        </div>
        <div id="Num_ass">
            <label for="num_assigments">Number Assignments:</label>
            <input type="number" id="num_assigments" name="num" value="<?php echo $num_grdes ?>">
        </div>
        <div id="Half_life">
            <label for="half">Half-life:</label>
            <input type="number" id="half" name="half" value="<?php echo $half_life ?>">
        </div>
        <div id="Improve">
            <label for="inc">Simulate Improvement:</label>
            <input type="checkbox" id="inc" name="inc" <?php if ($increase) echo 'checked' ?> ></input>
        </div>
        <div id="Above">
            <label for="inc">Above 50%:</label>
            <input type="checkbox" id="fifty" name="fifty" <?php if ($above_50) echo 'checked' ?> ></input>
        </div>
        <div class="button">
               
            <button type="submit">Update</button>
             
        </div>
    </form>


<?php

echo $graphics->getSvg(1300, 1000);



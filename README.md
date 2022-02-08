'modules' => [
    'admin' => [
        'class' => 'app\modules\admin\Module',
    ],
    'main' => [
        'class' => 'app\modules\main\Module',
    ],
    'user' => [
        'class' => 'app\modules\user\Module',
    ],
],
//  модуль admin также, как мы собирали модуль user

// Интернационализация
$this->title = $model->id;
$this->params['breadcrumbs'][] = ['label' => Yii::t('app', 'Users'), 'url' => ['index']];
$this->params['breadcrumbs'][] = $this->title;
$this->title = $model->username;
$this->params['breadcrumbs'][] = ['label' => Yii::t('app', 'ADMIN_USERS'), 'url' => ['index']];
$this->params['breadcrumbs'][] = $this->title;

 переводы в наши языковые файлы в папке messages.
 
 // Навигация
 echo Nav::widget([
    'options' => ['class' => 'navbar-nav navbar-right'],
    'activateParents' => true,
    'items' => array_filter([
        ['label' => Yii::t('app', 'NAV_HOME'), 'url' => ['/main/default/index']],
        ['label' => Yii::t('app', 'NAW_CONTACT'), 'url' => ['/main/contact/index']],
        Yii::$app->user->isGuest ?
            ['label' => Yii::t('app', 'NAV_SIGNUP'), 'url' => ['/user/default/signup']] :
            false,
        Yii::$app->user->isGuest ?
            ['label' => Yii::t('app', 'NAV_LOGIN'), 'url' => ['/user/default/login']] :
            false,
        !Yii::$app->user->isGuest ?
            ['label' => Yii::t('app', 'NAV_ADMIN'), 'items' => [
                ['label' => Yii::t('app', 'NAV_ADMIN'), 'url' => ['/admin/default/index']],
                ['label' => Yii::t('app', 'ADMIN_USERS'), 'url' => ['/admin/users/index']],
            ]] :
            false,
        !Yii::$app->user->isGuest ?
            ['label' => Yii::t('app', 'NAV_PROFILE'), 'items' => [
                ['label' => Yii::t('app', 'NAV_PROFILE'), 'url' => ['/user/profile/index']],
                ['label' => Yii::t('app', 'NAV_LOGOUT'),
                    'url' => ['/user/default/logout'],
                    'linkOptions' => ['data-method' => 'post']]
            ]] :
            false,
    ]),
]);

modules/admin/views/default/index.php удалим текстовую заглушку и тоже вставим ссылку:
<?php
<?php
 
use yii\helpers\Html;
 
/* @var $this yii\web\View */
/* @var $model \app\modules\user\models\User */
 
$this->title = Yii::t('app', 'ADMIN');
$this->params['breadcrumbs'][] = $this->title;
?>
<div class="admin-default-index">
    <h1><?= Html::encode($this->title) ?></h1>
 
    <p>
        <?= Html::a(Yii::t('app', 'ADMIN_USERS'), ['users/index'], ['class' => 'btn btn-primary']) ?>
    </p>
</div>

      /перейдём к функционалу./
      
      namespace app\modules\admin\models;
 
use yii\helpers\ArrayHelper;
use Yii;
 
class User extends \app\modules\user\models\User
{
    const SCENARIO_ADMIN_CREATE = 'adminCreate';
    const SCENARIO_ADMIN_UPDATE = 'adminUpdate';
 
    public $newPassword;
    public $newPasswordRepeat;
 
    public function rules()
    {
        return ArrayHelper::merge(parent::rules(), [
            [['newPassword', 'newPasswordRepeat'], 'required', 'on' => self::SCENARIO_ADMIN_CREATE],
            ['newPassword', 'string', 'min' => 6],
            ['newPasswordRepeat', 'compare', 'compareAttribute' => 'newPassword'],
        ]);
    }
 
    public function scenarios()
    {
        $scenarios = parent::scenarios();
        $scenarios[self::SCENARIO_ADMIN_CREATE] = ['username', 'email', 'status', 'newPassword', 'newPasswordRepeat'];
        $scenarios[self::SCENARIO_ADMIN_UPDATE] = ['username', 'email', 'status', 'newPassword', 'newPasswordRepeat'];
        return $scenarios;
    }
 
    public function attributeLabels()
    {
        return ArrayHelper::merge(parent::attributeLabels(), [
            'newPassword' => Yii::t('app', 'USER_NEW_PASSWORD'),
            'newPasswordRepeat' => Yii::t('app', 'USER_REPEAT_PASSWORD'),
        ]);
    }
 
    public function beforeSave($insert)
    {
        if (parent::beforeSave($insert)) {
            if (!empty($this->newPassword)) {
                $this->setPassword($this->newPassword);
            }
            return true;
        }
        return false;
    }
}

// в модуле admin
use app\modules\user\models\User; == use app\modules\admin\models\User;

//  сценарии в действиях контроллера:

namespace app\modules\admin\controllers;
 
use Yii;
use app\modules\admin\models\User;
use app\modules\admin\models\UserSearch;
use yii\web\Controller;
use yii\web\NotFoundHttpException;
use yii\filters\VerbFilter;
 
/**
 * UsersController implements the CRUD actions for User model.
 */
class UsersController extends Controller
{
    ...
 
    public function actionCreate()
    {
        $model = new User();
        $model->scenario = User::SCENARIO_ADMIN_CREATE;
        $model->status = User::STATUS_ACTIVE;
 
        if ($model->load(Yii::$app->request->post()) && $model->save()) {
            return $this->redirect(['view', 'id' => $model->id]);
        } else {
            return $this->render('create', [
                'model' => $model,
            ]);
        }
    }
 
    public function actionUpdate($id)
    {
        $model = $this->findModel($id);
        $model->scenario = User::SCENARIO_ADMIN_UPDATE;
 
        if ($model->load(Yii::$app->request->post()) && $model->save()) {
            return $this->redirect(['view', 'id' => $model->id]);
        } else {
            return $this->render('update', [
                'model' => $model,
            ]);
        }
    }
}

// В modules/admin/views/users/view.php выведем статус пользователя названием, а не числом.

<?php
<?= DetailView::widget([
    'model' => $model,
    'attributes' => [
        'id',
        'username',
        'email:email',
        'created_at:datetime',
        'updated_at:datetime',
        [
            'attribute' => 'status',
            'value' => $model->getStatusName(),
        ],
    ],
]) ?>

<?php
<?php
 
use yii\helpers\Html;
 
/* @var $this yii\web\View */
/* @var $model app\modules\admin\models\User */
 
$this->title = $model->username;
$this->params['breadcrumbs'][] = ['label' => Yii::t('app', 'ADMIN'), 'url' => ['default/index']];
$this->params['breadcrumbs'][] = ['label' => Yii::t('app', 'ADMIN_USERS'), 'url' => ['index']];
$this->params['breadcrumbs'][] = ['label' => $model->username, 'url' => ['view', 'id' => $model->id]];
$this->params['breadcrumbs'][] = Yii::t('app', 'TITLE_UPDATE');
?>
<div class="user-update">
 
    <h1><?= Html::encode($this->title) ?></h1>
 
    <?= $this->render('_form', [
        'model' => $model,
    ]) ?>
 
</div>

В modules/admin/views/users/_form.php добавляем поля для нового пароля и выпадающий список с вариантами для статуса:

<?php
<?php
 
use app\modules\admin\models\User;
use yii\helpers\Html;
use yii\widgets\ActiveForm;
 
/* @var $this yii\web\View */
/* @var $model app\modules\admin\models\User */
/* @var $form yii\widgets\ActiveForm */
?>
 
<div class="user-form">
 
    <?php $form = ActiveForm::begin(); ?>
 
    <?= $form->field($model, 'username')->textInput(['maxlength' => true]) ?>
 
    <?= $form->field($model, 'email')->textInput(['maxlength' => true]) ?>
 
    <?= $form->field($model, 'newPassword')->passwordInput(['maxlength' => true]) ?>
 
    <?= $form->field($model, 'newPasswordRepeat')->passwordInput(['maxlength' => true]) ?>
 
    <?= $form->field($model, 'status')->dropDownList(User::getStatusesArray()) ?>
 
    <div class="form-group">
        <?= Html::submitButton(
            $model->isNewRecord ? Yii::t('app', 'BUTTON_CREATE') : Yii::t('app', 'BUTTON_CREATE'),
            ['class' => $model->isNewRecord ? 'btn btn-success' : 'btn btn-primary']
        ) ?>
    </div>
 
    <?php ActiveForm::end(); ?>
 
</div>

// Поисковая модель 
namespace app\modules\admin\models;
 
use Yii;
use yii\base\Model;
use yii\data\ActiveDataProvider;
 
class UserSearch extends Model
{
    public $id;
    public $username;
    public $email;
    public $status;
 
    public function rules()
    {
        return [
            [['id', 'status'], 'integer'],
            [['username', 'email'], 'safe'],
        ];
    }
 
    public function attributeLabels()
    {
        return [
            'id' => 'ID',
            'created_at' => Yii::t('app', 'USER_CREATED'),
            'updated_at' => Yii::t('app', 'USER_UPDATED'),
            'username' => Yii::t('app', 'USER_USERNAME'),
            'email' => Yii::t('app', 'USER_EMAIL'),
            'status' => Yii::t('app', 'USER_STATUS'),
        ];
    }
 
    public function search($params)
    {
        $query = User::find();
 
        $dataProvider = new ActiveDataProvider([
            'query' => $query,
        ]);
 
        $this->load($params);
 
        if (!$this->validate()) {
            $query->where('0=1');
            return $dataProvider;
        }
 
        $query->andFilterWhere([
            'id' => $this->id,
            'status' => $this->status,
        ]);
 
        $query
            ->andFilterWhere(['like', 'username', $this->username])
            ->andFilterWhere(['like', 'email', $this->email]);
 
        return $dataProvider;
    }
}

// Сортировка 
class UserSearch extends Model
{
    ...
 
    public function search($params)
    {
        $query = User::find();
 
        $dataProvider = new ActiveDataProvider([
            'query' => $query,
            'sort' => [
                'defaultOrder' => ['id' => SORT_DESC],
            ],
        ]);
 
        ...
    }
}

[
    'class' => 'yii\grid\ActionColumn',
    'contentOptions' => ['style' => 'white-space: nowrap; text-align: center; letter-spacing: 0.1em; max-width: 7em;'],
],

// отдельный класс колонки:
namespace app\components\grid;
 
use yii\grid\ActionColumn;
 
class ActionColumn extends ActionColumn
{
    public $contentOptions = [
        'style' => 'white-space: nowrap; text-align: center; letter-spacing: 0.1em; max-width: 7em;',
    ];
}
// или 
namespace app\components\grid;
 
class ActionColumn extends \yii\grid\ActionColumn
{
    public $contentOptions = [
        'class' => 'action-column',
    ];
}

// колонка статуса
<?php
<?= GridView::widget([
    'dataProvider' => $dataProvider,
    'filterModel' => $searchModel,
    'columns' => [
        'id',
        'created_at:datetime',
        'username',
        'email:email',
        [
            'filter' => User::getStatusesArray(),
            'attribute' => 'status',
            'format' => 'raw',
            'value' => function ($model, $key, $index, $column) {
                /** @var User $model */
                /** @var \yii\grid\DataColumn $column */
                $value = $model->{$column->attribute};
                switch ($value) {
                    case User::STATUS_ACTIVE:
                        $class = 'success';
                        break;
                    case User::STATUS_WAIT:
                        $class = 'warning';
                        break;
                    case User::STATUS_BLOCKED:
                    default:
                        $class = 'default';
                };
                $html = Html::tag('span', Html::encode($model->getStatusName()), ['class' => 'label label-' . $class]);
                return $value === null ? $column->grid->emptyCell : $html;
            }
        ],
        ['class' => ActionColumn::className()],
    ],
]); ?>

// сделаем универсальную колонку для статусов и прочих полей с наборами вариантов. Для этого полностью отвяжем компонент от модели пользователя. Для этого помимо основного поля attribute добавим ещё одно настраиваемое поле name типа callable для получения наименования статуса

namespace app\components\grid;
 
use yii\grid\DataColumn;
use yii\helpers\ArrayHelper;
use yii\helpers\Html;
 
class SetColumn extends DataColumn
{
    /**
     * @var callable
     */
    public $name;
    /**
     * Array of status classes
     * ```
     * [
     *     User::STATUS_ACTIVE => 'success',
     *     User::STATUS_WAIT => 'warning',
     *     User::STATUS_BLOCKED => 'default',
     * ]
     * ```
     * @var array
     */
    public $cssCLasses = [];
 
    protected function renderDataCellContent($model, $key, $index)
    {
        $value = $this->getDataCellValue($model, $key, $index);
        $name = $this->getStatusName($model, $key, $index, $value);
        $class = ArrayHelper::getValue($this->cssCLasses, $value, 'default');
        $html = Html::tag('span', Html::encode($name), ['class' => 'label label-' . $class]);
        return $value === null ? $this->grid->emptyCell : $html;
    }
 
    /**
     * @param mixed $model
     * @param mixed $key
     * @param integer $index
     * @param mixed $value
     * @return string
     */
    private function getStatusName($model, $key, $index, $value)
    {
        if ($this->name !== null) {
            if (is_string($this->name)) {
                $name = ArrayHelper::getValue($model, $this->name);
            } else {
                $name = call_user_func($this->name, $model, $key, $index, $this);
            }
        } else {
            $name = null;
        }
        return $name === null ? $value : $name;
    }
}

// Поиск по дате. Добавим для этого в поисковую модель два поля $date_from и $date_to

namespace app\modules\admin\models;
 
use Yii;
use yii\base\Model;
use yii\data\ActiveDataProvider;
 
/**
 * UserSearch represents the model behind the search form about `app\modules\admin\models\User`.
 */
class UserSearch extends Model
{
    public $id;
    public $username;
    public $email;
    public $status;
    public $date_from;
    public $date_to;
 
    public function rules()
    {
        return [
            [['id', 'status'], 'integer'],
            [['username', 'email'], 'safe'],
            [['date_from', 'date_to'], 'date', 'format' => 'php:Y-m-d'],
        ];
    }
    ...
}

// эффективнее преобразовать наши даты в числа функцией strtotime и воспользоваться простыми неравенствами без вычислений на стороне SQL:
         //  наша поисковая модель:
namespace app\modules\admin\models;
 
use Yii;
use yii\base\Model;
use yii\data\ActiveDataProvider;
 
/**
 * UserSearch represents the model behind the search form about `app\modules\admin\models\User`.
 */
class UserSearch extends Model
{
    public $id;
    public $username;
    public $email;
    public $status;
    public $date_from;
    public $date_to;
 
    public function rules()
    {
        return [
            [['id', 'status'], 'integer'],
            [['username', 'email'], 'safe'],
            [['date_from', 'date_to'], 'date', 'format' => 'php:Y-m-d'],
        ];
    }
 
    public function attributeLabels()
    {
        return [
            'id' => 'ID',
            'created_at' => Yii::t('app', 'USER_CREATED'),
            'updated_at' => Yii::t('app', 'USER_UPDATED'),
            'username' => Yii::t('app', 'USER_USERNAME'),
            'email' => Yii::t('app', 'USER_EMAIL'),
            'status' => Yii::t('app', 'USER_STATUS'),
            'date_from' => Yii::t('app', 'USER_DATE_FROM'),
            'date_to' => Yii::t('app', 'USER_DATE_TO'),
        ];
    }
 
    /**
     * Creates data provider instance with search query applied
     *
     * @param array $params
     *
     * @return ActiveDataProvider
     */
    public function search($params)
    {
        $query = User::find();
 
        $dataProvider = new ActiveDataProvider([
            'query' => $query,
            'sort' => [
                'defaultOrder' => ['id' => SORT_DESC],
            ]
        ]);
 
        $this->load($params);
 
        if (!$this->validate()) {
            $query->where('0=1');
            return $dataProvider;
        }
 
        $query->andFilterWhere([
            'id' => $this->id,
            'status' => $this->status,
        ]);
 
        $query
            ->andFilterWhere(['like', 'username', $this->username])
            ->andFilterWhere(['like', 'email', $this->email])
            ->andFilterWhere(['>=', 'created_at', $this->date_from ? strtotime($this->date_from . ' 00:00:00') : null])
            ->andFilterWhere(['<=', 'created_at', $this->date_to ? strtotime($this->date_to . ' 23:59:59') : null]);
 
        return $dataProvider;
    }
}

// Имя пользователя ссылкой
namespace app\components\grid;
 
use Closure;
use yii\grid\DataColumn;
use yii\helpers\Html;
use yii\helpers\Url;
 
class LinkColumn extends DataColumn
{
    /**
     * @var callable
     */
    public $url;
    /**
     * @var bool
     */
    public $targetBlank = false;
    /**
     * @var string
     */
    public $controller;
    /**
     * @inheritdoc
     */
    public $format = 'raw';
 
    protected function renderDataCellContent($model, $key, $index)
    {
        $value = $this->getDataCellValue($model, $key, $index);
        $text = $this->grid->formatter->format($value, $this->format);
        $url = $this->createUrl($model, $key, $index);
        $options = $this->targetBlank ? ['target' => '_blank'] : [];
        return $value === null ? $this->grid->emptyCell : Html::a($text, $url, $options);
    }
 
    public function createUrl($model, $key, $index)
    {
        if ($this->url instanceof Closure) {
            return call_user_func($this->url, $model, $key, $index);
        } else {
            $params = is_array($key) ? $key : ['id' => (string) $key];
            $params[0] = $this->controller ? $this->controller . '/view' : 'view';
            return Url::toRoute($params);
        }
    }
    
   // Контроль доступа
   namespace app\modules\admin;
 
use yii\filters\AccessControl;
 
class Module extends \yii\base\Module
{
    public $controllerNamespace = 'app\modules\admin\controllers';
 
    public function behaviors()
    {
        return [
            'access' => [
                'class' => AccessControl::className(),
                'rules' => [
                    [
                        'allow' => true,
                        'roles' => ['@'],
                    ],
                ],
            ],
        ];
    }
}
